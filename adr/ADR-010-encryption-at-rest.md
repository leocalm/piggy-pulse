# ADR-010: Encryption at Rest

## Context

PiggyPulse stores sensitive financial data: transaction amounts and descriptions, account balances, category names, vendor names, subscription billing amounts, and budget targets. A server-side breach or database compromise would expose this data if stored in plaintext.

The platform already protects data in transit (HTTPS/TLS), authenticates users (Argon2 password hashing, HttpOnly session cookies, Bearer tokens), and supports optional 2FA. However, none of these defences protect the data at the storage layer. If an attacker gains read access to the PostgreSQL database — through a SQL injection vulnerability, a compromised backup, or an infrastructure-level breach — all user financial data would be readable.

Encryption at rest ensures that data on disk is ciphertext, not plaintext. Decryption requires a per-user Data Encryption Key (DEK) that is not stored on the server in usable form.

## Decision

- **Algorithm:** AES-256-GCM (AES-GCM with 256-bit key, 96-bit nonce, 128-bit authentication tag).
- **Key hierarchy:**
  - Each user has a 32-byte DEK (Data Encryption Key) generated from a CSPRNG at signup.
  - The DEK is wrapped (encrypted) using a KEK (Key Encryption Key) derived from the user's password via Argon2id.
  - The wrapped DEK (`wrapped_dek`) and wrap parameters (`dek_wrap_params` — Argon2id salt/costs + AES-GCM wrap nonce) are stored on the server alongside the user record.
  - The plaintext DEK is held only in client memory between unwrap and upload, and in a server-side per-session store for the lifetime of the authenticated session.
- **DEK transport:**
  - After login, the client receives `wrapped_dek` and `dek_wrap_params` in the login response.
  - The client derives the KEK locally via Argon2id(password, salt), unwraps the DEK, and POSTs the plaintext DEK (base64-encoded) to `POST /v2/auth/unlock`.
  - The server stores the DEK in an in-process session store (`SessionDekStore`) keyed by the session or API token principal ID.
  - On logout or session expiry, the DEK is removed from the session store.
- **Encryption scope:** All user-data columns across these tables are encrypted:

  | Table | Encrypted columns |
  |---|---|
  | `transaction` | `amount_enc`, `description_enc` |
  | `logical_transaction_state` | `current_sum_enc` |
  | `account` | `current_balance_enc`, `name_enc`, `color_enc`, `icon_enc`, `spend_limit_enc`, `next_transfer_amount_enc`, `top_up_amount_enc` |
  | `category` | `name_enc`, `color_enc`, `icon_enc`, `description_enc` |
  | `vendor` | `name_enc`, `description_enc` |
  | `budget_category` | `budgeted_value_enc` |
  | `subscription` | `name_enc`, `billing_amount_enc` |

- **Client-side encryption:** Both the web app (React/TypeScript) and iOS client implement the full encryption stack — Argon2id KEK derivation, DEK unwrap, AES-256-GCM encrypt/decrypt of server responses. This means sensitive data is encrypted before it reaches the database and decrypted only in the client.
- **Key protection:** The `Dek` Rust type zeroizes on drop, omits `Debug`, `Serialize`, `Clone`, and `Display` derives. The only sanctioned copy path is `clone_for_request()`, with a deliberately awkward name to discourage casual use.

## Alternatives Considered

- **PostgreSQL `pgcrypto` extension:** Would encrypt at the database level but would not protect against a server-side process with DB read access. The key would still live on the server, reducing the security model to "trust the server process."
- **Application-level encryption with server-held key:** Simpler to implement (single key stored in an env var or HSM), but a server compromise would expose all users' data. Per-user keys raise the cost of a mass breach.
- **Cloud KMS (AWS KMS, GCP Cloud KMS):** Would provide managed key storage and rotation, but introduces cloud dependency, adds latency to every read/write, and complicates the self-hosted deployment story.
- **Transparent Data Encryption (TDE):** Encrypts the entire database at the filesystem or block level. Protects against stolen disks or backups but not against a live DB read query from a compromised application process.

## Consequences

### Positive
- **Defence in depth:** Even with full database read access, an attacker cannot recover plaintext financial data without each user's password.
- **Client-side trust model:** The server never holds the plaintext DEK in persistent storage — only in per-session memory.
- **User-controlled key material:** The DEK is derived from the user's password; changing the password re-wraps the DEK under a new KEK.
- **Open source verification:** The encryption implementation is fully visible in the API, web, and iOS repositories.

### Negative
- **No server-side aggregation:** The ledger refactor's materialized aggregate tables (`user_daily_totals`, `category_all_time`, etc.) were dropped because the server can no longer sum ciphertext. Dashboard aggregates are computed client-side.
- **Increased login complexity:** Users must complete the unlock flow after login (DEK unwrap + POST). This adds a step to the authentication sequence.
- **No DB-level uniqueness enforcement:** Unique constraints on encrypted columns (e.g. `account_user_id_name_key`) were dropped because AES-GCM produces different ciphertext per encryption. Uniqueness is enforced in the service layer by decrypting existing names.
- **Session store is in-process (v1):** The current `SessionDekStore` uses an in-process `HashMap`. A server restart or horizontal scale-out would lose all session DEKs, requiring clients to re-unlock. A Redis-backed store is planned for Phase 5.
- **Performance overhead:** Every write encrypts and every read decrypts the encrypted fields. For a single-user or small-scale deployment the overhead is negligible; at scale, the per-field ciphertext expansion (up to ~48 bytes per field due to GCM tag and nonce) increases storage and bandwidth.
- **No encryption for Android client:** The Android app (currently marked as outdated) does not implement the encryption flow. Users on Android would not be able to decrypt their data.

## Reversibility

- **Low.** The migration is destructive — it truncates all user data before altering columns. Reverting requires restoring from a pre-migration backup or re-populating data. The down-migration restores the schema but does not recover encrypted data.

## Related Documents

- [`SECURITY.md`](../SECURITY.md) – overall security design and principles
- [`.kiro/specs/encryption-at-rest/design.md`](https://github.com/leocalm/piggy-pulse-api/blob/main/.kiro/specs/encryption-at-rest/design.md) – full design document
- [`src/crypto.rs`](https://github.com/leocalm/piggy-pulse-api/blob/main/src/crypto.rs) – DEK and AES-GCM primitives
- [`src/session_dek.rs`](https://github.com/leocalm/piggy-pulse-api/blob/main/src/session_dek.rs) – per-session DEK store
- [`src/routes/v2/auth/unlock.rs`](https://github.com/leocalm/piggy-pulse-api/blob/main/src/routes/v2/auth/unlock.rs) – DEK unlock endpoint
- [`migrations/20260327000008_encryption_at_rest.up.sql`](https://github.com/leocalm/piggy-pulse-api/blob/main/migrations/20260327000008_encryption_at_rest.up.sql) – schema migration
