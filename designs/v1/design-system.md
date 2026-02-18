PiggyPulse Design System v1

Version: 1.0
Last Updated: 2026-02-16
Scope: Dashboard and Categories
Purpose: Establish visual, structural, and behavioral consistency across the product.

⸻

1. Core Philosophy

PiggyPulse is a reflective financial clarity platform.

The interface must:
	•	Be descriptive, not advisory.
	•	Encode magnitude, not morality.
	•	Preserve neutrality.
	•	Avoid gamification.
	•	Reduce stress, not increase it.
	•	Feel warm and human without being childish.

Language Rules

Use:
	•	Variance
	•	Percent used
	•	Projected at current pace
	•	Stability

Avoid:
	•	Over
	•	Under
	•	Warning
	•	Alert
	•	On track
	•	Good
	•	Bad
	•	Overspending
	•	Celebratory language

⸻

2. Page Archetypes

PiggyPulse uses two primary page types.

⸻

2.1 Reflective Page (Dashboard)

Purpose:
Am I stable?

Characteristics:
	•	Larger numbers
	•	More breathing room
	•	Subtle glow allowed
	•	Fewer elements
	•	Centered hero layout
	•	Longitudinal emphasis (alignment over time)

Visual Tone:
Soft, reflective, calm.

⸻

2.2 Diagnostic Page (Categories)

Purpose:
Where is pressure?

Characteristics:
	•	Higher density
	•	Structured rows
	•	Clear metric hierarchy
	•	Spatial encoding (bars)
	•	No glow
	•	Stronger typographic clarity

Visual Tone:
Precise, analytical, grounded.

⸻

3. Foundations

⸻

3.1 Color Tokens

Brand Accents:

Accent Primary: #5E63E6
Accent Secondary: #8C6CFB

Used for:
	•	Progress fills
	•	Stability dots (outside tolerance)
	•	Gradients
	•	Key data signals

Never used for:
	•	Emotional feedback
	•	Red/green states
	•	Success/failure cues

⸻

3.2 Surface Tokens

Light Mode:

Background: #F4F6FB
Surface: #FFFFFF
Border Soft: #E6E8F0

Text Primary: #1A1D21
Text Secondary: #5C6470
Text Muted: #8B93A1

Dark Mode:

Background: #14161A
Surface: #1C1F26
Border Soft: #2A2F38

Text Primary: #E6E8EB
Text Secondary: #A8AFB9
Text Muted: #7D8590

Rules:
	•	Dashboard may use subtle glow.
	•	Diagnostic pages remain flatter.
	•	Accent colors remain consistent across themes.

⸻

3.3 Typography Scale

Font: Inter

Dashboard Scale:

Hero value: 28 to 32px, weight 700
Secondary value: 20px, weight 700
Labels: 13 to 14px, weight 500 to 600

Diagnostic Scale:

Category name: 15px, weight 600
Metric value: 13px, weight 650
Variance value: 13px, weight 800
Projection: 12px, weight 400 to 500
Context labels: 11px, weight 500

Hierarchy rule:

Variance is strongest.
Budget and Actual are medium weight.
Labels are muted.
Projection is secondary.

No color-based emphasis.

⸻

4. Structural Components

⸻

4.1 Context Strip (Global Top Bar)

Used on:
	•	Dashboard
	•	Categories
	•	Future pages

Structure:

Period Selector
Elapsed Bar
Summary Numbers

Rules:
	•	Sticky
	•	Elapsed bar thinner than page bars
	•	Contextual, not focal
	•	No animation
	•	No glow

⸻

4.2 Progress System

Two variants.

Reflective Progress (Dashboard):
	•	8px height
	•	No overflow
	•	May animate subtly
	•	May use soft glow

Diagnostic Progress (Categories):
	•	6px track
	•	Visible 100 percent boundary marker
	•	40px capped overflow zone
	•	No glow
	•	No animation
	•	Geometry communicates overflow

Rules:
	•	100 percent ends exactly at track boundary
	•	Above 100 percent extends into overflow zone
	•	Overflow visually capped
	•	No red/green encoding

⸻

4.3 Stability Indicator

Used in:
	•	Alignment (Dashboard)
	•	Category rows

Structure:
Dot-based indicator representing last 3 closed periods.

Rules:
	•	Filled dot equals outside tolerance
	•	Empty dot equals within tolerance
	•	Slight size difference allowed for clarity
	•	No color change by direction
	•	Label must clarify meaning:
Stability (last 3 closed periods)

⸻

4.4 Metric Column Pattern

Structure:

Label aligned left
Value aligned right

Rules:
	•	Values right-aligned
	•	Minimum width for value column
	•	Variance strongest weight
	•	Labels muted
	•	Horizontal spacing between label and value
	•	Slightly increased vertical rhythm

No inline explanation words such as “over budget”.

⸻

4.5 Unbudgeted Row Pattern

Structure:

Category on left
Dollar amount and share percent on right

Rules:
	•	Sorted descending by spend
	•	No variance
	•	No projection
	•	No progress bar
	•	Clean separation from budgeted section

⸻

5. Interaction Principles

Hover (Desktop):

Allowed:
	•	Subtle elevation
	•	Slight border tint
	•	Soft shadow

Not allowed:
	•	Color flipping
	•	Dramatic animation
	•	State-based color changes

Motion:

Allowed:
	•	Subtle number transitions
	•	Soft fades
	•	Gentle gradient shifts on Dashboard only

Not allowed:
	•	Bounce
	•	Confetti
	•	Success or failure animation
	•	Gamified effects

PiggyPulse is reflective, not gamified.

⸻

6. Visual Encoding Rules

Allowed:
	•	Spatial encoding
	•	Weight contrast
	•	Scale contrast
	•	Subtle gradient
	•	Direction arrows (up and down)

Forbidden:
	•	Red/green states
	•	Emotional colors
	•	Emoji celebration
	•	“On track”
	•	“Warning”
	•	Alert states

Magnitude does not equal morality.

⸻

7. Layout System

Max width: 1100px
Centered content.

Diagnostic pages use a 2 to 1 grid layout for category rows.
Reflective pages use centered hero and stacked cards.

Diagnostic pages are denser.
Reflective pages are more spacious.

⸻

8. Information Architecture Flow

User flow:

Dashboard
Categories
Transactions
Accounts

Abstraction decreases as user moves deeper.

Dashboard: high-level stability
Categories: structured variance
Transactions: raw detail
Accounts: balance-level structure

Each page must:
	•	Increase granularity
	•	Maintain neutrality
	•	Preserve tone consistency

⸻

9. Component Inventory (v1)

ContextStrip
HeroCard
AlignmentCard
NetPositionCard
ReflectiveProgressTrack
DiagnosticProgressTrack
MetricColumn
StabilityDots
UnbudgetedRow
ThemeToggle

⸻

10. Guardrails

When designing a new component, ask:
	1.	Does this describe or judge?
	2.	Does it encode magnitude without moral tone?
	3.	Is hierarchy achieved through structure, not color?
	4.	Is the user treated as a rational adult?
	5.	Would this increase stress?

If any answer violates philosophy, redesign.

⸻

End of v1.
