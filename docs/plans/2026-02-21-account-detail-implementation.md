# Account Detail Page Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build the account detail page (`/accounts/:accountId`) with four new backend endpoints and a progressive-loading React frontend.

**Architecture:** Four focused backend endpoints under `/api/v1/accounts/<id>/` (detail, balance-history, transactions, context), each with its own model, DB method, and route handler. The frontend loads the header/chart/transactions immediately and defers the context section until the user expands it.

**Tech Stack:** Backend — Rust, Rocket, SQLx, PostgreSQL. Frontend — React, TypeScript, Mantine UI, Mantine Charts (`AreaChart`), React Query.

---

## Repo layout

```
piggy-pulse-api/          ← backend
  src/
    models/account.rs     ← response structs (Serialize + JsonSchema)
    database/account.rs   ← PostgresRepository impl methods + SQL
    routes/account.rs     ← Rocket route handlers + tests

piggy-pulse-app/          ← frontend
  src/
    api/accounts.ts       ← API client functions
    hooks/useAccount*.ts  ← React Query hooks
    pages/AccountDetail/  ← page + sub-components
```

> **Note:** Frontend paths follow the existing convention in piggy-pulse-app. Check `src/pages/` and `src/api/` for exact naming before creating files.

---

## Task 1: `AccountDetailResponse` model + DB method + route

**Purpose:** Power the page header and period flow section.

### Files
- Modify: `piggy-pulse-api/src/models/account.rs`
- Modify: `piggy-pulse-api/src/database/account.rs`
- Modify: `piggy-pulse-api/src/routes/account.rs`

### Step 1: Add the response struct to `src/models/account.rs`

Append after the last struct in the file:

```rust
#[derive(Serialize, Debug, JsonSchema)]
pub struct AccountDetailResponse {
    pub balance: i64,
    pub balance_change: i64,
    pub inflows: i64,
    pub outflows: i64,
    pub net: i64,
    pub period_start: NaiveDate,
    pub period_end: NaiveDate,
}
```

Ensure `NaiveDate` is imported: `use chrono::NaiveDate;`

### Step 2: Add the DB method to `src/database/account.rs`

Append a new method inside the existing `impl PostgresRepository` block:

```rust
pub async fn get_account_detail(
    &self,
    account_id: &Uuid,
    period_id: &Uuid,
    user_id: &Uuid,
) -> Result<AccountDetailResponse, AppError> {
    #[derive(sqlx::FromRow)]
    struct DetailRow {
        balance: i64,
        inflows: i64,
        outflows: i64,
        period_start: NaiveDate,
        period_end: NaiveDate,
    }

    let row = sqlx::query_as::<_, DetailRow>(
        r#"
WITH period AS (
    SELECT start_date, end_date
    FROM budget_period
    WHERE id = $2 AND user_id = $3
),
period_txs AS (
    SELECT
        t.amount,
        c.category_type,
        t.from_account_id,
        t.to_account_id
    FROM transaction t
    JOIN category c ON c.id = t.category_id
    CROSS JOIN period
    WHERE t.user_id = $3
      AND (t.from_account_id = $1 OR t.to_account_id = $1)
      AND t.occurred_at >= period.start_date
      AND t.occurred_at <= period.end_date
),
flow AS (
    SELECT
        COALESCE(SUM(CASE
            WHEN category_type = 'Incoming' THEN amount
            WHEN category_type = 'Transfer' AND to_account_id = $1 THEN amount
            ELSE 0
        END), 0) AS inflows,
        COALESCE(SUM(CASE
            WHEN category_type = 'Outgoing' THEN amount
            WHEN category_type = 'Transfer' AND from_account_id = $1 THEN amount
            ELSE 0
        END), 0) AS outflows
    FROM period_txs
)
SELECT
    a.balance                         AS balance,
    flow.inflows::bigint              AS inflows,
    flow.outflows::bigint             AS outflows,
    period.start_date                 AS period_start,
    period.end_date                   AS period_end
FROM account a
CROSS JOIN flow
CROSS JOIN period
WHERE a.id = $1 AND a.user_id = $3
        "#,
    )
    .bind(account_id)
    .bind(period_id)
    .bind(user_id)
    .fetch_optional(&self.pool)
    .await?
    .ok_or_else(|| AppError::NotFound("Account not found".to_string()))?;

    let net = row.inflows - row.outflows;
    let balance_change = net; // net flow equals balance change for the period
    Ok(AccountDetailResponse {
        balance: row.balance,
        balance_change,
        inflows: row.inflows,
        outflows: row.outflows,
        net,
        period_start: row.period_start,
        period_end: row.period_end,
    })
}
```

Add to imports at top of `database/account.rs` if missing: `use chrono::NaiveDate;`

### Step 3: Add the route handler to `src/routes/account.rs`

```rust
/// Get account detail metrics for a budget period
#[openapi(tag = "Accounts")]
#[get("/<id>/detail?<period_id>")]
pub async fn get_account_detail(
    pool: &State<PgPool>,
    _rate_limit: RateLimit,
    current_user: CurrentUser,
    id: &str,
    period_id: String,
) -> Result<Json<AccountDetailResponse>, AppError> {
    let repo = PostgresRepository { pool: pool.inner().clone() };
    let account_uuid = Uuid::parse_str(id).map_err(|e| AppError::uuid("Invalid account id", e))?;
    let period_uuid = Uuid::parse_str(&period_id).map_err(|e| AppError::uuid("Invalid period id", e))?;
    let detail = repo.get_account_detail(&account_uuid, &period_uuid, &current_user.id).await?;
    Ok(Json(detail))
}
```

Add `AccountDetailResponse` to the models import at the top of `routes/account.rs`.

Register in `routes()`:
```rust
rocket_okapi::openapi_get_routes_spec![
    // ... existing routes ...
    get_account_detail,
]
```

### Step 4: Write the integration test

Inside the `#[cfg(test)] mod tests` block in `routes/account.rs`, add:

```rust
#[rocket::async_test]
#[ignore = "requires database"]
async fn test_get_account_detail() {
    let mut config = Config::default();
    config.database.url = "postgres://postgres:example@127.0.0.1:5432/piggy_pulse_db".to_string();
    config.rate_limit.require_client_ip = false;
    config.session.cookie_secure = false;

    let client = Client::tracked(build_rocket(config)).await.expect("valid rocket instance");
    create_user_and_auth(&client).await;

    let category_id = create_category(&client, "Salary", "Incoming").await;
    let expense_id = create_category(&client, "Rent", "Outgoing").await;
    let account_id = create_account(&client, "Test Checking", 100_000).await;

    let today = Utc::now().date_naive();
    let start = today.checked_sub_signed(Duration::days(5)).unwrap_or(today);
    let period_id = create_budget_period(&client, start, today).await;

    create_transaction(&client, &category_id, &account_id, start, 50_000).await;
    create_transaction(&client, &expense_id, &account_id, start, 20_000).await;

    let response = client
        .get(format!("/api/v1/accounts/{}/detail?period_id={}", account_id, period_id))
        .dispatch()
        .await;
    assert_eq!(response.status(), Status::Ok);

    let body = response.into_string().await.expect("response body");
    let json: Value = serde_json::from_str(&body).expect("valid json");

    assert_eq!(json["inflows"].as_i64().unwrap(), 50_000);
    assert_eq!(json["outflows"].as_i64().unwrap(), 20_000);
    assert_eq!(json["net"].as_i64().unwrap(), 30_000);
}
```

### Step 5: Run the test

```bash
cd piggy-pulse-api
cargo test test_get_account_detail -- --ignored
```

Expected: PASS (requires running DB; skip with `-- --ignored` in CI).

Also verify it compiles cleanly:
```bash
cargo clippy --workspace --all-targets -- -D warnings
```

### Step 6: Commit

```bash
git add src/models/account.rs src/database/account.rs src/routes/account.rs
git commit -m "feat(accounts): add GET /accounts/:id/detail endpoint"
```

---

## Task 2: Balance history endpoint

**Purpose:** Power the `AreaChart` for Period / 30d / 90d / 1y views.

### Files
- Modify: `piggy-pulse-api/src/models/account.rs`
- Modify: `piggy-pulse-api/src/database/account.rs`
- Modify: `piggy-pulse-api/src/routes/account.rs`

### Step 1: Add response struct to `src/models/account.rs`

```rust
#[derive(Serialize, Debug, JsonSchema)]
pub struct AccountBalanceHistoryPoint {
    pub date: String,    // "YYYY-MM-DD"
    pub balance: i64,    // integer cents
}
```

### Step 2: Add DB method to `src/database/account.rs`

The query is a generalisation of `list_account_balance_per_day`. It accepts a `(start_date, end_date)` window instead of a period UUID for calendar ranges.

```rust
pub async fn get_account_balance_history(
    &self,
    account_id: &Uuid,
    start_date: NaiveDate,
    end_date: NaiveDate,
    user_id: &Uuid,
) -> Result<Vec<AccountBalanceHistoryPoint>, AppError> {
    #[derive(sqlx::FromRow)]
    struct HistoryRow {
        date: String,
        balance: i64,
    }

    let rows = sqlx::query_as::<_, HistoryRow>(
        r#"
WITH days AS (
    SELECT d::date AS day
    FROM generate_series($2::date, $3::date, '1 day') AS d
),
base_balance AS (
    SELECT
        a.balance + COALESCE(SUM(
            CASE
                WHEN c.category_type = 'Incoming'                              THEN  t.amount::bigint
                WHEN c.category_type = 'Outgoing'                              THEN -t.amount::bigint
                WHEN c.category_type = 'Transfer' AND t.from_account_id = a.id THEN -t.amount::bigint
                WHEN c.category_type = 'Transfer' AND t.to_account_id   = a.id THEN  t.amount::bigint
                ELSE 0
            END
        ), 0) AS base_bal
    FROM account a
    LEFT JOIN transaction t ON (t.from_account_id = a.id OR t.to_account_id = a.id)
                            AND t.occurred_at < $2
                            AND t.user_id = $4
    LEFT JOIN category c ON t.category_id = c.id
    WHERE a.id = $1 AND a.user_id = $4
    GROUP BY a.id, a.balance
),
daily_totals AS (
    SELECT
        t.occurred_at::date AS day,
        SUM(
            CASE
                WHEN c.category_type = 'Incoming'                              THEN  t.amount::bigint
                WHEN c.category_type = 'Outgoing'                              THEN -t.amount::bigint
                WHEN c.category_type = 'Transfer' AND t.from_account_id = $1  THEN -t.amount::bigint
                WHEN c.category_type = 'Transfer' AND t.to_account_id   = $1  THEN  t.amount::bigint
                ELSE 0
            END
        ) AS daily_amount
    FROM transaction t
    JOIN category c ON c.id = t.category_id
    WHERE (t.from_account_id = $1 OR t.to_account_id = $1)
      AND t.user_id = $4
      AND t.occurred_at >= $2
      AND t.occurred_at <= $3
    GROUP BY t.occurred_at::date
)
SELECT
    to_char(d.day, 'YYYY-MM-DD') AS date,
    (bb.base_bal + SUM(COALESCE(dt.daily_amount, 0)) OVER (
        ORDER BY d.day
        ROWS UNBOUNDED PRECEDING
    ))::bigint AS balance
FROM days d
CROSS JOIN base_balance bb
LEFT JOIN daily_totals dt ON dt.day = d.day
ORDER BY d.day
        "#,
    )
    .bind(account_id)
    .bind(start_date)
    .bind(end_date)
    .bind(user_id)
    .fetch_all(&self.pool)
    .await?;

    Ok(rows.into_iter().map(|r| AccountBalanceHistoryPoint { date: r.date, balance: r.balance }).collect())
}
```

### Step 3: Add route to `src/routes/account.rs`

```rust
/// Get balance history for an account
#[openapi(tag = "Accounts")]
#[get("/<id>/balance-history?<range>&<period_id>")]
pub async fn get_account_balance_history(
    pool: &State<PgPool>,
    _rate_limit: RateLimit,
    current_user: CurrentUser,
    id: &str,
    range: String,
    period_id: Option<String>,
) -> Result<Json<Vec<AccountBalanceHistoryPoint>>, AppError> {
    let repo = PostgresRepository { pool: pool.inner().clone() };
    let account_uuid = Uuid::parse_str(id).map_err(|e| AppError::uuid("Invalid account id", e))?;

    let today = chrono::Utc::now().date_naive();
    let (start, end) = match range.as_str() {
        "30d"    => (today - chrono::Duration::days(30), today),
        "90d"    => (today - chrono::Duration::days(90), today),
        "1y"     => (today - chrono::Duration::days(365), today),
        "period" => {
            let pid = period_id.ok_or_else(|| AppError::BadRequest("period_id required for range=period".to_string()))?;
            let period_uuid = Uuid::parse_str(&pid).map_err(|e| AppError::uuid("Invalid period id", e))?;
            let period = repo.get_budget_period(&period_uuid, &current_user.id).await?;
            (period.start_date, period.end_date)
        }
        _ => return Err(AppError::BadRequest("Invalid range. Use: period, 30d, 90d, 1y".to_string())),
    };

    let points = repo.get_account_balance_history(&account_uuid, start, end, &current_user.id).await?;
    Ok(Json(points))
}
```

> **Note:** Check what `get_budget_period` returns in `src/database/budget_period.rs` — you need access to `start_date` and `end_date`. Adjust field names if they differ.

Register in `routes()` and add `AccountBalanceHistoryPoint` to the models import.

### Step 4: Write integration test

```rust
#[rocket::async_test]
#[ignore = "requires database"]
async fn test_get_account_balance_history_period() {
    // ... setup as in task 1 ...
    let response = client
        .get(format!("/api/v1/accounts/{}/balance-history?range=period&period_id={}", account_id, period_id))
        .dispatch()
        .await;
    assert_eq!(response.status(), Status::Ok);
    let json: Value = serde_json::from_str(&response.into_string().await.unwrap()).unwrap();
    assert!(json.as_array().unwrap().len() > 0);
    let point = &json[0];
    assert!(point["date"].as_str().is_some());
    assert!(point["balance"].as_i64().is_some());
}

#[rocket::async_test]
#[ignore = "requires database"]
async fn test_get_account_balance_history_invalid_range() {
    // ... setup ...
    let response = client
        .get(format!("/api/v1/accounts/{}/balance-history?range=bad", account_id))
        .dispatch()
        .await;
    assert_eq!(response.status(), Status::BadRequest);
}
```

### Step 5: Run and verify

```bash
cargo clippy --workspace --all-targets -- -D warnings
cargo test test_get_account_balance_history -- --ignored
```

### Step 6: Commit

```bash
git add src/models/account.rs src/database/account.rs src/routes/account.rs
git commit -m "feat(accounts): add GET /accounts/:id/balance-history endpoint"
```

---

## Task 3: Account transactions endpoint

**Purpose:** Transaction list filtered by account, with running balance.

### Files
- Modify: `piggy-pulse-api/src/models/account.rs`
- Modify: `piggy-pulse-api/src/database/account.rs`
- Modify: `piggy-pulse-api/src/routes/account.rs`

### Step 1: Add response struct

```rust
#[derive(Serialize, Debug, JsonSchema)]
pub struct AccountTransactionResponse {
    pub id: Uuid,
    pub amount: i64,
    pub description: String,
    pub occurred_at: NaiveDate,
    pub category_name: String,
    pub category_color: String,
    pub flow: String,            // "in" | "out"
    pub running_balance: i64,    // cents, server-computed
}
```

### Step 2: Add DB method

```rust
pub async fn get_account_transactions(
    &self,
    account_id: &Uuid,
    period_id: &Uuid,
    flow_filter: Option<&str>, // None = all, Some("in"), Some("out")
    params: &CursorParams,
    user_id: &Uuid,
) -> Result<Vec<AccountTransactionResponse>, AppError> {
    #[derive(sqlx::FromRow)]
    struct TxRow {
        id: Uuid,
        amount: i64,
        description: String,
        occurred_at: NaiveDate,
        category_name: String,
        category_color: String,
        flow: String,
        running_balance: i64,
    }

    let flow_clause = match flow_filter {
        Some("in")  => "AND flow = 'in'",
        Some("out") => "AND flow = 'out'",
        _           => "",
    };

    let cursor_clause = if let Some(cursor) = &params.cursor {
        format!("AND (t.occurred_at, t.id) < ('{}'::date, '{}'::uuid)", cursor.date, cursor.id)
    } else {
        String::new()
    };

    // Use a raw query with format! only for the safe static clauses above.
    // The actual user data ($1–$4) is always parameterised.
    let sql = format!(
        r#"
WITH period AS (
    SELECT start_date, end_date FROM budget_period WHERE id = $2 AND user_id = $4
),
base_balance AS (
    SELECT
        a.balance + COALESCE(SUM(
            CASE
                WHEN c.category_type = 'Incoming'                              THEN  t.amount::bigint
                WHEN c.category_type = 'Outgoing'                              THEN -t.amount::bigint
                WHEN c.category_type = 'Transfer' AND t.from_account_id = a.id THEN -t.amount::bigint
                WHEN c.category_type = 'Transfer' AND t.to_account_id   = a.id THEN  t.amount::bigint
                ELSE 0
            END
        ), 0) AS base_bal
    FROM account a
    LEFT JOIN transaction t  ON (t.from_account_id = a.id OR t.to_account_id = a.id)
                             AND t.occurred_at < (SELECT start_date FROM period)
                             AND t.user_id = $4
    LEFT JOIN category c ON t.category_id = c.id
    WHERE a.id = $1 AND a.user_id = $4
    GROUP BY a.id, a.balance
),
period_txs AS (
    SELECT
        t.id,
        t.amount::bigint AS amount,
        t.description,
        t.occurred_at,
        cat.name  AS category_name,
        cat.color AS category_color,
        CASE
            WHEN cat.category_type = 'Incoming'                              THEN 'in'
            WHEN cat.category_type = 'Transfer' AND t.to_account_id = $1    THEN 'in'
            ELSE 'out'
        END AS flow
    FROM transaction t
    JOIN category cat ON cat.id = t.category_id
    CROSS JOIN period
    WHERE (t.from_account_id = $1 OR t.to_account_id = $1)
      AND t.user_id = $4
      AND t.occurred_at >= period.start_date
      AND t.occurred_at <= period.end_date
),
with_running AS (
    SELECT
        pt.*,
        (bb.base_bal + SUM(
            CASE WHEN flow = 'in' THEN amount ELSE -amount END
        ) OVER (ORDER BY occurred_at ASC, id ASC ROWS UNBOUNDED PRECEDING))::bigint AS running_balance
    FROM period_txs pt
    CROSS JOIN base_balance bb
)
SELECT * FROM with_running
WHERE 1=1 {flow_clause} {cursor_clause}
ORDER BY occurred_at DESC, id DESC
LIMIT $3
        "#,
        flow_clause = flow_clause,
        cursor_clause = cursor_clause,
    );

    let rows = sqlx::query_as::<_, TxRow>(&sql)
        .bind(account_id)
        .bind(period_id)
        .bind(params.effective_limit() as i64)
        .bind(user_id)
        .fetch_all(&self.pool)
        .await?;

    Ok(rows.into_iter().map(|r| AccountTransactionResponse {
        id: r.id,
        amount: r.amount,
        description: r.description,
        occurred_at: r.occurred_at,
        category_name: r.category_name,
        category_color: r.category_color,
        flow: r.flow,
        running_balance: r.running_balance,
    }).collect())
}
```

> **Note on the cursor:** Check how `CursorParams` stores the cursor value (look at `src/models/pagination.rs`). If the cursor is an opaque base64 string encoding `(date, id)`, adapt accordingly — you may need to decode it first. If it's just a UUID, simplify to `t.id < $cursor`.

### Step 3: Add route

```rust
/// List transactions for a specific account within a budget period
#[openapi(tag = "Accounts")]
#[get("/<id>/transactions?<period_id>&<type>&<cursor>&<limit>")]
pub async fn list_account_transactions(
    pool: &State<PgPool>,
    _rate_limit: RateLimit,
    current_user: CurrentUser,
    id: &str,
    period_id: String,
    r#type: Option<String>,
    cursor: Option<String>,
    limit: Option<i64>,
) -> Result<Json<CursorPaginatedResponse<AccountTransactionResponse>>, AppError> {
    let repo = PostgresRepository { pool: pool.inner().clone() };
    let account_uuid = Uuid::parse_str(id).map_err(|e| AppError::uuid("Invalid account id", e))?;
    let period_uuid = Uuid::parse_str(&period_id).map_err(|e| AppError::uuid("Invalid period id", e))?;
    let params = CursorParams::from_query(cursor, limit)?;

    let flow_filter = r#type.as_deref();
    let txs = repo.get_account_transactions(&account_uuid, &period_uuid, flow_filter, &params, &current_user.id).await?;
    Ok(Json(CursorPaginatedResponse::from_rows(txs, params.effective_limit(), |r| r.id)))
}
```

Register in `routes()`. Add `AccountTransactionResponse` to model imports.

### Step 4: Write integration test

```rust
#[rocket::async_test]
#[ignore = "requires database"]
async fn test_list_account_transactions() {
    // ... setup user, account, period, transactions ...
    let response = client
        .get(format!("/api/v1/accounts/{}/transactions?period_id={}", account_id, period_id))
        .dispatch()
        .await;
    assert_eq!(response.status(), Status::Ok);
    let json: Value = serde_json::from_str(&response.into_string().await.unwrap()).unwrap();
    let data = json["data"].as_array().unwrap();
    // Verify running_balance field is present
    assert!(data[0]["running_balance"].as_i64().is_some());
    assert!(data[0]["flow"].as_str().is_some());
}
```

### Step 5: Run and verify

```bash
cargo clippy --workspace --all-targets -- -D warnings
cargo test test_list_account_transactions -- --ignored
```

### Step 6: Commit

```bash
git add src/models/account.rs src/database/account.rs src/routes/account.rs
git commit -m "feat(accounts): add GET /accounts/:id/transactions endpoint"
```

---

## Task 4: Account context endpoint

**Purpose:** Category impact + stability stats, loaded lazily on expand.

### Files
- Modify: `piggy-pulse-api/src/models/account.rs`
- Modify: `piggy-pulse-api/src/database/account.rs`
- Modify: `piggy-pulse-api/src/routes/account.rs`

### Step 1: Add response structs

```rust
#[derive(Serialize, Debug, JsonSchema)]
pub struct CategoryImpactItem {
    pub category_name: String,
    pub amount: i64,
    pub percentage: i32,
}

#[derive(Serialize, Debug, JsonSchema)]
pub struct AccountStability {
    pub periods_closed_positive: i64,
    pub periods_evaluated: i64,
    pub avg_closing_balance: i64,
    pub highest_closing_balance: i64,
    pub lowest_closing_balance: i64,
    pub largest_single_outflow: i64,
    pub largest_single_outflow_category: String,
}

#[derive(Serialize, Debug, JsonSchema)]
pub struct AccountContextResponse {
    pub category_impact: Vec<CategoryImpactItem>,
    pub stability: AccountStability,
}
```

### Step 2: Add DB method

```rust
pub async fn get_account_context(
    &self,
    account_id: &Uuid,
    period_id: &Uuid,
    user_id: &Uuid,
) -> Result<AccountContextResponse, AppError> {
    // --- Category impact: outflows in current period, grouped by category ---
    #[derive(sqlx::FromRow)]
    struct CategoryRow {
        category_name: String,
        amount: i64,
    }

    let cat_rows = sqlx::query_as::<_, CategoryRow>(
        r#"
WITH period AS (
    SELECT start_date, end_date FROM budget_period WHERE id = $2 AND user_id = $3
)
SELECT
    cat.name AS category_name,
    SUM(t.amount)::bigint AS amount
FROM transaction t
JOIN category cat ON cat.id = t.category_id
CROSS JOIN period
WHERE t.from_account_id = $1
  AND t.user_id = $3
  AND cat.category_type = 'Outgoing'
  AND t.occurred_at >= period.start_date
  AND t.occurred_at <= period.end_date
GROUP BY cat.name
ORDER BY amount DESC
        "#,
    )
    .bind(account_id)
    .bind(period_id)
    .bind(user_id)
    .fetch_all(&self.pool)
    .await?;

    let total_outflows: i64 = cat_rows.iter().map(|r| r.amount).sum();
    let category_impact = cat_rows.into_iter().map(|r| {
        let pct = if total_outflows > 0 { (r.amount * 100 / total_outflows) as i32 } else { 0 };
        CategoryImpactItem { category_name: r.category_name, amount: r.amount, percentage: pct }
    }).collect();

    // --- Stability: look at up to 6 prior closed periods ---
    #[derive(sqlx::FromRow)]
    struct StabilityRow {
        closing_balance: i64,
        is_positive: bool,
    }

    // For each prior period, compute the account's closing balance
    // A period is "closed" if end_date < today
    let stability_rows = sqlx::query_as::<_, StabilityRow>(
        r#"
WITH prior_periods AS (
    SELECT id, start_date, end_date
    FROM budget_period
    WHERE user_id = $2
      AND end_date < CURRENT_DATE
      AND id != $3
    ORDER BY end_date DESC
    LIMIT 6
),
period_flows AS (
    SELECT
        pp.id AS period_id,
        pp.end_date,
        COALESCE(SUM(
            CASE
                WHEN c.category_type = 'Incoming'                              THEN  t.amount::bigint
                WHEN c.category_type = 'Outgoing'                              THEN -t.amount::bigint
                WHEN c.category_type = 'Transfer' AND t.from_account_id = $1  THEN -t.amount::bigint
                WHEN c.category_type = 'Transfer' AND t.to_account_id   = $1  THEN  t.amount::bigint
                ELSE 0
            END
        ), 0) AS net_flow
    FROM prior_periods pp
    LEFT JOIN transaction t ON (t.from_account_id = $1 OR t.to_account_id = $1)
                            AND t.user_id = $2
                            AND t.occurred_at >= pp.start_date
                            AND t.occurred_at <= pp.end_date
    LEFT JOIN category c ON c.id = t.category_id
    GROUP BY pp.id, pp.end_date
),
base AS (
    SELECT a.balance AS current_balance FROM account a WHERE a.id = $1 AND a.user_id = $2
)
SELECT
    (base.current_balance + SUM(pf.net_flow) OVER (ORDER BY pf.end_date DESC ROWS UNBOUNDED PRECEDING))::bigint AS closing_balance,
    ((base.current_balance + SUM(pf.net_flow) OVER (ORDER BY pf.end_date DESC ROWS UNBOUNDED PRECEDING)) > 0) AS is_positive
FROM period_flows pf
CROSS JOIN base
        "#,
    )
    .bind(account_id)
    .bind(user_id)
    .bind(period_id)
    .fetch_all(&self.pool)
    .await?;

    let periods_evaluated = stability_rows.len() as i64;
    let periods_closed_positive = stability_rows.iter().filter(|r| r.is_positive).count() as i64;
    let balances: Vec<i64> = stability_rows.iter().map(|r| r.closing_balance).collect();
    let avg = if periods_evaluated > 0 { balances.iter().sum::<i64>() / periods_evaluated } else { 0 };
    let highest = balances.iter().copied().max().unwrap_or(0);
    let lowest = balances.iter().copied().min().unwrap_or(0);

    // Largest single outflow in current period
    #[derive(sqlx::FromRow)]
    struct OutflowRow {
        category_name: String,
        amount: i64,
    }

    let largest = sqlx::query_as::<_, OutflowRow>(
        r#"
WITH period AS (
    SELECT start_date, end_date FROM budget_period WHERE id = $2 AND user_id = $3
)
SELECT
    cat.name AS category_name,
    t.amount::bigint AS amount
FROM transaction t
JOIN category cat ON cat.id = t.category_id
CROSS JOIN period
WHERE t.from_account_id = $1
  AND t.user_id = $3
  AND cat.category_type = 'Outgoing'
  AND t.occurred_at >= period.start_date
  AND t.occurred_at <= period.end_date
ORDER BY t.amount DESC
LIMIT 1
        "#,
    )
    .bind(account_id)
    .bind(period_id)
    .bind(user_id)
    .fetch_optional(&self.pool)
    .await?;

    let (largest_outflow, largest_outflow_cat) = largest
        .map(|r| (r.amount, r.category_name))
        .unwrap_or((0, String::new()));

    Ok(AccountContextResponse {
        category_impact,
        stability: AccountStability {
            periods_closed_positive,
            periods_evaluated,
            avg_closing_balance: avg,
            highest_closing_balance: highest,
            lowest_closing_balance: lowest,
            largest_single_outflow: largest_outflow,
            largest_single_outflow_category: largest_outflow_cat,
        },
    })
}
```

### Step 3: Add route

```rust
/// Get context (category impact + stability) for an account — loaded lazily
#[openapi(tag = "Accounts")]
#[get("/<id>/context?<period_id>")]
pub async fn get_account_context(
    pool: &State<PgPool>,
    _rate_limit: RateLimit,
    current_user: CurrentUser,
    id: &str,
    period_id: String,
) -> Result<Json<AccountContextResponse>, AppError> {
    let repo = PostgresRepository { pool: pool.inner().clone() };
    let account_uuid = Uuid::parse_str(id).map_err(|e| AppError::uuid("Invalid account id", e))?;
    let period_uuid = Uuid::parse_str(&period_id).map_err(|e| AppError::uuid("Invalid period id", e))?;
    let ctx = repo.get_account_context(&account_uuid, &period_uuid, &current_user.id).await?;
    Ok(Json(ctx))
}
```

Register in `routes()`.

### Step 4: Write test

```rust
#[rocket::async_test]
#[ignore = "requires database"]
async fn test_get_account_context() {
    // ... setup ...
    let response = client
        .get(format!("/api/v1/accounts/{}/context?period_id={}", account_id, period_id))
        .dispatch()
        .await;
    assert_eq!(response.status(), Status::Ok);
    let json: Value = serde_json::from_str(&response.into_string().await.unwrap()).unwrap();
    assert!(json["category_impact"].as_array().is_some());
    assert!(json["stability"]["periods_evaluated"].as_i64().is_some());
}
```

### Step 5: Run and verify

```bash
cargo clippy --workspace --all-targets -- -D warnings
cargo test test_get_account_context -- --ignored
```

### Step 6: Commit

```bash
git add src/models/account.rs src/database/account.rs src/routes/account.rs
git commit -m "feat(accounts): add GET /accounts/:id/context endpoint"
```

---

## Task 5: Frontend — API client functions

**Purpose:** Typed fetch functions for the four new endpoints.

### Files
- Modify: `piggy-pulse-app/src/api/accounts.ts` (or create if it doesn't exist; check for the existing accounts API file)

### Step 1: Verify existing file location

```bash
find piggy-pulse-app/src/api -name "account*"
```

### Step 2: Add types and API functions

```typescript
// Types matching backend response shapes

export interface AccountDetail {
  balance: number;
  balanceChange: number;   // camelCase — client auto-converts from snake_case
  inflows: number;
  outflows: number;
  net: number;
  periodStart: string;
  periodEnd: string;
}

export interface BalanceHistoryPoint {
  date: string;
  balance: number;
}

export interface AccountTransaction {
  id: string;
  amount: number;
  description: string;
  occurredAt: string;
  categoryName: string;
  categoryColor: string;
  flow: 'in' | 'out';
  runningBalance: number;
}

export interface CategoryImpactItem {
  categoryName: string;
  amount: number;
  percentage: number;
}

export interface AccountStability {
  periodsClosedPositive: number;
  periodsEvaluated: number;
  avgClosingBalance: number;
  highestClosingBalance: number;
  lowestClosingBalance: number;
  largestSingleOutflow: number;
  largestSingleOutflowCategory: string;
}

export interface AccountContext {
  categoryImpact: CategoryImpactItem[];
  stability: AccountStability;
}

// API functions — use the existing `client` from src/api/client.ts

export const getAccountDetail = (accountId: string, periodId: string) =>
  client.get<AccountDetail>(`/accounts/${accountId}/detail`, { params: { period_id: periodId } });

export const getAccountBalanceHistory = (
  accountId: string,
  range: 'period' | '30d' | '90d' | '1y',
  periodId?: string,
) =>
  client.get<BalanceHistoryPoint[]>(`/accounts/${accountId}/balance-history`, {
    params: { range, ...(periodId ? { period_id: periodId } : {}) },
  });

export const getAccountTransactions = (
  accountId: string,
  periodId: string,
  type: 'all' | 'in' | 'out' = 'all',
  cursor?: string,
) =>
  client.get<{ data: AccountTransaction[]; nextCursor: string | null }>(
    `/accounts/${accountId}/transactions`,
    { params: { period_id: periodId, type, ...(cursor ? { cursor } : {}) } },
  );

export const getAccountContext = (accountId: string, periodId: string) =>
  client.get<AccountContext>(`/accounts/${accountId}/context`, { params: { period_id: periodId } });
```

> **Note:** The `client` import path and call signature depend on your existing API client. Check `src/api/client.ts` and match the pattern used by other API files.

### Step 3: Verify TypeScript compiles

```bash
cd piggy-pulse-app
yarn typecheck
```

### Step 4: Commit

```bash
git add src/api/accounts.ts
git commit -m "feat(accounts): add API client functions for account detail endpoints"
```

---

## Task 6: React Query hooks

**Purpose:** Wrap API functions with React Query for caching and loading states.

### Files
- Create: `piggy-pulse-app/src/hooks/useAccountDetail.ts`
- Create: `piggy-pulse-app/src/hooks/useAccountBalanceHistory.ts`
- Create: `piggy-pulse-app/src/hooks/useAccountTransactions.ts`
- Create: `piggy-pulse-app/src/hooks/useAccountContext.ts`

> Check the existing hooks directory — hooks may live in `src/hooks/` or co-located with pages. Follow the existing convention.

### Step 1: `useAccountDetail.ts`

```typescript
import { useQuery } from '@tanstack/react-query';
import { getAccountDetail } from '../api/accounts';

export const useAccountDetail = (accountId: string, periodId: string) =>
  useQuery({
    queryKey: ['account-detail', accountId, periodId],
    queryFn: () => getAccountDetail(accountId, periodId),
    enabled: Boolean(accountId && periodId),
  });
```

### Step 2: `useAccountBalanceHistory.ts`

```typescript
import { useQuery } from '@tanstack/react-query';
import { getAccountBalanceHistory } from '../api/accounts';

export const useAccountBalanceHistory = (
  accountId: string,
  range: 'period' | '30d' | '90d' | '1y',
  periodId?: string,
) =>
  useQuery({
    queryKey: ['account-balance-history', accountId, range, periodId],
    queryFn: () => getAccountBalanceHistory(accountId, range, periodId),
    enabled: Boolean(accountId) && (range !== 'period' || Boolean(periodId)),
  });
```

### Step 3: `useAccountTransactions.ts`

```typescript
import { useInfiniteQuery } from '@tanstack/react-query';
import { getAccountTransactions } from '../api/accounts';

export const useAccountTransactions = (
  accountId: string,
  periodId: string,
  type: 'all' | 'in' | 'out' = 'all',
) =>
  useInfiniteQuery({
    queryKey: ['account-transactions', accountId, periodId, type],
    queryFn: ({ pageParam }) => getAccountTransactions(accountId, periodId, type, pageParam as string | undefined),
    getNextPageParam: (last) => last.nextCursor ?? undefined,
    initialPageParam: undefined,
    enabled: Boolean(accountId && periodId),
  });
```

### Step 4: `useAccountContext.ts`

```typescript
import { useQuery } from '@tanstack/react-query';
import { getAccountContext } from '../api/accounts';

export const useAccountContext = (accountId: string, periodId: string, enabled: boolean) =>
  useQuery({
    queryKey: ['account-context', accountId, periodId],
    queryFn: () => getAccountContext(accountId, periodId),
    enabled: enabled && Boolean(accountId && periodId),
  });
```

The `enabled` flag is controlled by the parent — set to `true` only on first expand.

### Step 5: Verify

```bash
yarn typecheck
```

### Step 6: Commit

```bash
git add src/hooks/useAccountDetail.ts src/hooks/useAccountBalanceHistory.ts \
        src/hooks/useAccountTransactions.ts src/hooks/useAccountContext.ts
git commit -m "feat(accounts): add React Query hooks for account detail"
```

---

## Task 7: `AccountDetailHeader` + `PeriodFlow` components

### Files
- Create: `piggy-pulse-app/src/pages/AccountDetail/AccountDetailHeader.tsx`
- Create: `piggy-pulse-app/src/pages/AccountDetail/PeriodFlow.tsx`

### Step 1: `AccountDetailHeader.tsx`

```tsx
import { Skeleton, Text, Title } from '@mantine/core';
import { AccountDetail } from '../../api/accounts';
import { AccountResponse } from '../../api/accounts'; // existing type

interface Props {
  account: AccountResponse | undefined;
  detail: AccountDetail | undefined;
  isLoading: boolean;
}

export function AccountDetailHeader({ account, detail, isLoading }: Props) {
  if (isLoading) return <Skeleton height={60} />;

  const sign = detail && detail.balanceChange >= 0 ? '+' : '';
  const periodLabel = detail
    ? `${sign}${formatCents(detail.balanceChange)} · ${detail.periodStart} – ${detail.periodEnd}`
    : '';

  return (
    <div style={{ display: 'flex', justifyContent: 'space-between', flexWrap: 'wrap', gap: 20, marginBottom: 40 }}>
      <div>
        <Title order={4}>{account?.name}</Title>
        <Text size="xs" tt="uppercase" c="dimmed">{account?.accountType}</Text>
      </div>
      <div style={{ textAlign: 'right' }}>
        <Title order={2}>{formatCents(detail?.balance ?? 0)}</Title>
        <Text size="sm" c="dimmed">{periodLabel}</Text>
      </div>
    </div>
  );
}

function formatCents(cents: number): string {
  return new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(cents / 100);
}
```

> **Note:** Replace `formatCents` with your app's existing currency formatter if one exists.

### Step 2: `PeriodFlow.tsx`

```tsx
import { Skeleton, SimpleGrid, Text } from '@mantine/core';
import { AccountDetail } from '../../api/accounts';

interface Props {
  detail: AccountDetail | undefined;
  isLoading: boolean;
}

export function PeriodFlow({ detail, isLoading }: Props) {
  if (isLoading) return <Skeleton height={80} />;

  const rows = [
    { label: 'Inflows',  value: detail?.inflows  ?? 0 },
    { label: 'Outflows', value: detail?.outflows ?? 0 },
    { label: 'Net',      value: detail?.net       ?? 0, bold: true },
  ];

  return (
    <div style={{ marginBottom: 36 }}>
      <Text size="xs" tt="uppercase" c="dimmed" mb="xs">Period Flow</Text>
      {rows.map(({ label, value, bold }) => (
        <SimpleGrid key={label} cols={2} style={{ marginBottom: 4 }}>
          <Text fw={bold ? 700 : undefined}>{label}</Text>
          <Text ta="right" fw={bold ? 700 : undefined}>{fmt(value)}</Text>
        </SimpleGrid>
      ))}
    </div>
  );
}

const fmt = (cents: number) =>
  new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(cents / 100);
```

### Step 3: Verify

```bash
yarn typecheck
```

### Step 4: Commit

```bash
git add src/pages/AccountDetail/AccountDetailHeader.tsx src/pages/AccountDetail/PeriodFlow.tsx
git commit -m "feat(account-detail): add AccountDetailHeader and PeriodFlow components"
```

---

## Task 8: `BalanceHistoryChart` component

### Files
- Create: `piggy-pulse-app/src/pages/AccountDetail/BalanceHistoryChart.tsx`

### Step 1: Install Mantine Charts if not already present

```bash
cd piggy-pulse-app
yarn add @mantine/charts recharts
```

(Mantine Charts uses Recharts under the hood.)

### Step 2: Build the component

```tsx
import { useState } from 'react';
import { AreaChart } from '@mantine/charts';
import { Paper, SegmentedControl, Skeleton, Text } from '@mantine/core';
import { useAccountBalanceHistory } from '../../hooks/useAccountBalanceHistory';

type Range = 'period' | '30d' | '90d' | '1y';

interface Props {
  accountId: string;
  periodId: string;
}

export function BalanceHistoryChart({ accountId, periodId }: Props) {
  const [range, setRange] = useState<Range>('period');
  const { data, isLoading } = useAccountBalanceHistory(accountId, range, periodId);

  const chartData = (data ?? []).map((p) => ({
    date: p.date,
    Balance: p.balance / 100,
  }));

  return (
    <Paper withBorder p="md" mb="xl">
      <div style={{ display: 'flex', justifyContent: 'space-between', alignItems: 'center', marginBottom: 16 }}>
        <Text fw={500}>Balance History</Text>
        <SegmentedControl
          size="xs"
          value={range}
          onChange={(v) => setRange(v as Range)}
          data={[
            { label: 'Period', value: 'period' },
            { label: '30d',    value: '30d' },
            { label: '90d',    value: '90d' },
            { label: '1y',     value: '1y' },
          ]}
        />
      </div>
      {isLoading ? (
        <Skeleton height={260} />
      ) : (
        <AreaChart
          h={260}
          data={chartData}
          dataKey="date"
          series={[{ name: 'Balance', color: 'indigo.6' }]}
          curveType="linear"
          referenceLines={[{ y: 0, color: 'gray.4', label: '' }]}
          valueFormatter={(v) =>
            new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(v)
          }
        />
      )}
    </Paper>
  );
}
```

### Step 3: Verify

```bash
yarn typecheck
```

### Step 4: Commit

```bash
git add src/pages/AccountDetail/BalanceHistoryChart.tsx
git commit -m "feat(account-detail): add BalanceHistoryChart with range selector"
```

---

## Task 9: `AccountContextSection` component

### Files
- Create: `piggy-pulse-app/src/pages/AccountDetail/AccountContextSection.tsx`

### Step 1: Build the component (lazy-loaded on expand)

```tsx
import { useState } from 'react';
import { Collapse, Divider, Progress, SimpleGrid, Skeleton, Text, UnstyledButton } from '@mantine/core';
import { useAccountContext } from '../../hooks/useAccountContext';

interface Props {
  accountId: string;
  periodId: string;
}

export function AccountContextSection({ accountId, periodId }: Props) {
  const [opened, setOpened] = useState(false);
  const [everOpened, setEverOpened] = useState(false);

  const { data, isLoading } = useAccountContext(accountId, periodId, everOpened);

  const handleToggle = () => {
    setOpened((o) => !o);
    if (!everOpened) setEverOpened(true);
  };

  const maxAmount = data?.categoryImpact[0]?.amount ?? 1;

  return (
    <div>
      <Divider mb="md" />
      <UnstyledButton onClick={handleToggle}>
        <Text size="sm" tt="uppercase" c="dimmed">
          Context {opened ? '▴' : '▾'}
        </Text>
      </UnstyledButton>
      <Collapse in={opened}>
        {isLoading ? (
          <Skeleton height={200} mt="md" />
        ) : (
          <SimpleGrid cols={{ base: 1, sm: 2 }} spacing={56} mt="xl">
            <div>
              <Text size="xs" tt="uppercase" c="dimmed" mb="xs">Category Impact</Text>
              {data?.categoryImpact.map((item) => (
                <div key={item.categoryName} style={{ marginBottom: 12 }}>
                  <SimpleGrid cols={2}>
                    <Text size="sm">{item.categoryName}</Text>
                    <Text size="sm" ta="right">{fmt(item.amount)}</Text>
                  </SimpleGrid>
                  <Progress value={(item.amount / maxAmount) * 100} size="xs" mt={4} />
                </div>
              ))}
            </div>
            <div>
              <Text size="xs" tt="uppercase" c="dimmed" mb="xs">Stability</Text>
              {data && (
                <Text size="sm" c="dimmed" style={{ lineHeight: 1.8 }}>
                  Closed positive in {data.stability.periodsClosedPositive} of {data.stability.periodsEvaluated} periods<br />
                  Average closing balance: {fmt(data.stability.avgClosingBalance)}<br />
                  Highest closing balance: {fmt(data.stability.highestClosingBalance)}<br />
                  Lowest closing balance: {fmt(data.stability.lowestClosingBalance)}<br />
                  Largest single outflow: {fmt(data.stability.largestSingleOutflow)} ({data.stability.largestSingleOutflowCategory})
                </Text>
              )}
            </div>
          </SimpleGrid>
        )}
      </Collapse>
    </div>
  );
}

const fmt = (cents: number) =>
  new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(cents / 100);
```

### Step 2: Verify

```bash
yarn typecheck
```

### Step 3: Commit

```bash
git add src/pages/AccountDetail/AccountContextSection.tsx
git commit -m "feat(account-detail): add AccountContextSection with lazy loading"
```

---

## Task 10: `AccountTransactions` component

### Files
- Create: `piggy-pulse-app/src/pages/AccountDetail/AccountTransactions.tsx`

### Step 1: Build the component

```tsx
import { useState } from 'react';
import { Button, Group, Skeleton, Table, Text } from '@mantine/core';
import { useAccountTransactions } from '../../hooks/useAccountTransactions';

type FlowFilter = 'all' | 'in' | 'out';

interface Props {
  accountId: string;
  periodId: string;
}

export function AccountTransactions({ accountId, periodId }: Props) {
  const [type, setType] = useState<FlowFilter>('all');
  const { data, isLoading, fetchNextPage, hasNextPage, isFetchingNextPage } =
    useAccountTransactions(accountId, periodId, type);

  const transactions = data?.pages.flatMap((p) => p.data) ?? [];

  // Group by date
  const grouped = transactions.reduce<Record<string, typeof transactions>>((acc, tx) => {
    (acc[tx.occurredAt] ??= []).push(tx);
    return acc;
  }, {});

  return (
    <div style={{ marginTop: 60, borderTop: '1px solid var(--mantine-color-default-border)', paddingTop: 32 }}>
      <Text size="xs" tt="uppercase" c="dimmed" mb="md">Transactions</Text>

      <Group gap="xs" mb="md">
        {(['all', 'in', 'out'] as FlowFilter[]).map((f) => (
          <Button
            key={f}
            size="xs"
            variant={type === f ? 'outline' : 'subtle'}
            onClick={() => setType(f)}
          >
            {f === 'all' ? 'All' : f === 'in' ? 'Inflows' : 'Outflows'}
          </Button>
        ))}
      </Group>

      {isLoading ? (
        <Skeleton height={200} />
      ) : transactions.length === 0 ? (
        <Text c="dimmed" ta="center" py="xl">No transactions in this period.</Text>
      ) : (
        <Table>
          <Table.Thead>
            <Table.Tr>
              <Table.Th>Date</Table.Th>
              <Table.Th>Description</Table.Th>
              <Table.Th>Category</Table.Th>
              <Table.Th ta="right">Amount</Table.Th>
              <Table.Th ta="right">Balance</Table.Th>
            </Table.Tr>
          </Table.Thead>
          <Table.Tbody>
            {Object.entries(grouped).sort(([a], [b]) => b.localeCompare(a)).map(([date, txs]) => (
              <>
                <Table.Tr key={`group-${date}`}>
                  <Table.Td colSpan={5}>
                    <Text size="xs" tt="uppercase" c="dimmed">{date}</Text>
                  </Table.Td>
                </Table.Tr>
                {txs.map((tx) => (
                  <Table.Tr key={tx.id}>
                    <Table.Td>{tx.occurredAt}</Table.Td>
                    <Table.Td>{tx.description}</Table.Td>
                    <Table.Td>{tx.categoryName}</Table.Td>
                    <Table.Td ta="right" c={tx.flow === 'in' ? 'indigo' : undefined}>
                      {tx.flow === 'in' ? '+' : '-'}{fmt(tx.amount)}
                    </Table.Td>
                    <Table.Td ta="right" c="dimmed">{fmt(tx.runningBalance)}</Table.Td>
                  </Table.Tr>
                ))}
              </>
            ))}
          </Table.Tbody>
        </Table>
      )}

      {hasNextPage && (
        <Button variant="subtle" onClick={() => fetchNextPage()} loading={isFetchingNextPage} mt="md" fullWidth>
          Load more
        </Button>
      )}
    </div>
  );
}

const fmt = (cents: number) =>
  new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(cents / 100);
```

### Step 2: Verify

```bash
yarn typecheck
```

### Step 3: Commit

```bash
git add src/pages/AccountDetail/AccountTransactions.tsx
git commit -m "feat(account-detail): add AccountTransactions component with filter and pagination"
```

---

## Task 11: `AccountDetailPage` + route + navigation

**Purpose:** Assemble all components, register the route, wire navigation from accounts list.

### Files
- Create: `piggy-pulse-app/src/pages/AccountDetail/index.tsx`
- Modify: router file (check `src/router.tsx`, `src/App.tsx`, or `src/routes.tsx`)
- Modify: accounts list component (find it and add `onClick` navigating to `/accounts/:id`)

### Step 1: Create the page

```tsx
import { useParams } from 'react-router-dom';
import { Container } from '@mantine/core';
import { usePeriod } from '../../hooks/usePeriod'; // check actual hook name for current period
import { useAccount } from '../../hooks/useAccount'; // existing hook for GET /accounts/:id
import { useAccountDetail } from '../../hooks/useAccountDetail';
import { AccountDetailHeader } from './AccountDetailHeader';
import { BalanceHistoryChart } from './BalanceHistoryChart';
import { PeriodFlow } from './PeriodFlow';
import { AccountContextSection } from './AccountContextSection';
import { AccountTransactions } from './AccountTransactions';

export function AccountDetailPage() {
  const { accountId } = useParams<{ accountId: string }>();
  const { currentPeriodId } = usePeriod(); // adapt to actual period context API

  const { data: account, isLoading: accountLoading } = useAccount(accountId!);
  const { data: detail, isLoading: detailLoading } = useAccountDetail(accountId!, currentPeriodId!);

  return (
    <Container size="lg" py="xl">
      <AccountDetailHeader
        account={account}
        detail={detail}
        isLoading={accountLoading || detailLoading}
      />
      <BalanceHistoryChart accountId={accountId!} periodId={currentPeriodId!} />
      <PeriodFlow detail={detail} isLoading={detailLoading} />
      <AccountContextSection accountId={accountId!} periodId={currentPeriodId!} />
      <AccountTransactions accountId={accountId!} periodId={currentPeriodId!} />
    </Container>
  );
}
```

> **Note:** `usePeriod` / `currentPeriodId` — find the actual hook or context that provides the currently selected period ID in the app. Adapt the import accordingly.

### Step 2: Register the route

In your router file (e.g. `src/App.tsx` or `src/router.tsx`):

```tsx
import { AccountDetailPage } from './pages/AccountDetail';

// Inside your <Routes>:
<Route path="/accounts/:accountId" element={<AccountDetailPage />} />
```

### Step 3: Add navigation from accounts list

Find the accounts list component and add a click handler:

```tsx
import { useNavigate } from 'react-router-dom';

const navigate = useNavigate();

// On each account row/card:
onClick={() => navigate(`/accounts/${account.id}`)}
```

### Step 4: Run the full test suite

```bash
cd piggy-pulse-app
yarn test
```

Expected: all checks pass (typecheck + prettier + lint + vitest + build).

### Step 5: Commit

```bash
git add src/pages/AccountDetail/ src/App.tsx  # adjust paths as needed
git commit -m "feat(account-detail): assemble AccountDetailPage and register route"
```

---

## Task 12: Add DB indexes

**Purpose:** Ensure the new queries are fast in production.

### Step 1: Create the migration

```bash
cd piggy-pulse-api
sqlx migrate add account_detail_indexes
```

### Step 2: Write the migration

In the newly created `migrations/<timestamp>_account_detail_indexes.sql`:

```sql
-- Supports balance-history, detail, context, and transactions queries
-- All four endpoints query transactions by account and date
CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_transaction_account_date
    ON transaction (from_account_id, occurred_at);

CREATE INDEX CONCURRENTLY IF NOT EXISTS idx_transaction_to_account_date
    ON transaction (to_account_id, occurred_at);
```

> `CONCURRENTLY` avoids locking in production. It cannot run inside a transaction block — SQLx runs each migration in its own transaction, so you may need to add `-- sqlx:disable-transaction` at the top of the file if your SQLx version requires it. Check the existing migrations for the convention used.

### Step 3: Apply and verify

```bash
sqlx migrate run
```

### Step 4: Commit

```bash
git add migrations/
git commit -m "perf(db): add indexes on transaction(account, date) for account detail queries"
```

---

## Final verification

```bash
# Backend
cd piggy-pulse-api
cargo fmt
cargo clippy --workspace --all-targets -- -D warnings
cargo test

# Frontend
cd piggy-pulse-app
yarn test
```

Then smoke-test the full flow manually via Docker Compose or local dev servers.
