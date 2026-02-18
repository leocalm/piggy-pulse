
# LOCKED DASHBOARD SPEC
PiggyPulse — Structural State Transition Model

## 1. Concept

The Locked Dashboard replaces wizard-style onboarding.
Instead of guiding users through isolated steps, the main dashboard renders in a structurally locked state.

Cards unlock automatically when structural prerequisites are satisfied.

No gamification. No celebration. Pure structural reflection.

---

## 2. Structural Requirements

### Dashboard Unlock Requirements

- At least one valid period model defined
- At least one asset account created
- At least one Incoming category
- At least one Outgoing category

Unlocking is automatic and derived from system state.

---

## 3. Locked Card Visual Rules

Locked cards:
- 0.65 content opacity
- Desaturated surface background
- Dashed border (1px)
- No hover elevation
- No interactive pointer events

Locked cards contain:
- Title
- Requirement summary
- “Configure” CTA

---

## 4. Active Card Visual Rules

Active cards:
- Full opacity
- Standard surface background
- Solid border
- Hover enabled
- Full interaction enabled

---

## 5. Transition Rules

Trigger: Structural prerequisites satisfied.

Animation:
- Locked placeholder fades out (120ms, ease-out)
- Active content fades in (160ms, ease-in)

No movement, no scaling, no layout shift.

Simultaneous unlock if multiple cards qualify.

---

## 6. Micro Feedback (Optional)

Top banner:
“Structure complete. Dashboard unlocked.”

Duration: 3 seconds.
Neutral tone. Not celebratory.

---

## 7. Removal Behavior

If structural requirements become invalid:
Active → Locked transition uses same crossfade.
No warning animations.
System always reflects data.

---

## 8. Chart Rules

Charts render immediately with real data.
No animated-from-zero effects during unlock.
