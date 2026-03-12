---
name: startup-autopsy
description: Generate a visual HTML presentation diagnosing startup death risks. Input your industry + business model, get a stunning graveyard-themed deck with cross-industry failure patterns, real case studies with specific numbers, and survival strategies from 89 YC startups (2017-2025). Trigger on industry risk analysis, startup graveyard, failure patterns, or "how do startups in X die".
---

# Startup Autopsy

User inputs industry + business model → Identify their deadliest failure patterns → Generate a visual HTML presentation pulling the best case studies from the ENTIRE database (cross-industry), not just their category.

---

## Phase 1: Two-Layer Selection

Ask TWO questions in a SINGLE AskUserQuestion call:

**Question 1 — Industry** (header: "Industry"):
What industry is your product in?

Options:
- AI & Machine Learning — AI-first products, ML models, LLM wrappers, AI agents
- Fintech — Banking, payments, investing, lending, insurance, crypto
- Developer Tools — APIs, IDEs, infrastructure, DevOps, PaaS, observability
- Healthcare — Medical tech, biotech, drug discovery, health AI, genomics
- SaaS & B2B — Business software, productivity, security, sales tools, analytics
- Social & Community — Social networks, community, messaging, content, media
- Marketplace & Commerce — E-commerce, logistics, real estate, two-sided markets

**Question 2 — Business Model** (header: "Business Model"):
How does (or will) the product make money?

Options:
- B2C App — Consumer subscription, freemium, consumer-facing app
- B2B SaaS — Selling to businesses on recurring subscription
- API / Dev Platform — Usage-based pricing, developer-facing infrastructure
- Marketplace — Commission/take-rate on transactions between parties

If the user already specified both in their message, skip and go directly to Phase 2.

---

## Phase 2: Pattern Selection (THE CORE LOGIC)

Read [startup-database-v2.json](startup-database-v2.json) for company data.

### Step 1: Identify the user's top 3-4 patterns

Use the mapping below to find which patterns are deadliest for the user's industry × business model combo. Select the TOP 3-4 patterns.

#### Pattern Priority by Combo

**AI × B2B SaaS** → P1, P2, P11, P10
**AI × B2C App** → P2, P5, P8, P9
**AI × API/Dev Platform** → P2, P1, P9, P5
**Fintech × B2B SaaS** → P9, P13, P7, P10
**Fintech × B2C App** → P5, P9, P13, P2
**Fintech × API/Dev Platform** → P9, P2, P1, P8
**Fintech × Marketplace** → P8, P13, P3, P9
**Developer Tools × B2B SaaS** → P1, P9, P2, P10
**Developer Tools × API/Dev Platform** → P1, P2, P6, P5
**Healthcare × B2B SaaS** → P10, P9, P13, P3
**SaaS × B2B SaaS** → P1, P2, P11, P7
**SaaS × API/Dev Platform** → P2, P1, P9, P6
**Social × B2C App** → P12, P5, P9, P2
**Social × B2B SaaS** → P1, P2, P5, P12
**Marketplace × Marketplace** → P8, P12, P9, P1
**Marketplace × B2B SaaS** → P7, P1, P8, P9
**Marketplace × B2C App** → P1, P9, P8, P3

For combos not listed, pick the 3-4 patterns with the highest frequency in the user's industry.

### Step 2: For each selected pattern, pull the BEST case studies from the ENTIRE database

Do NOT limit to the user's industry. Pull the most compelling 3-5 cases per pattern from all 89 companies, prioritizing:
1. Cases with the most specific numbers (funding, users, revenue, timelines)
2. Cases with named killer competitors
3. Cases where the death_mechanism is most vivid
4. At least 1 case from the user's own industry (if one exists for that pattern)

This is what makes the presentation powerful: "This pattern killed an AI company, a fintech, and a dev tools company — here's how it will kill yours too."

### Step 3: For each pattern, identify 1-2 survivors from comparisons.similar_that_survived

Pull from the database entries. Prefer survivors that are well-known names the audience will recognize.

---

## Phase 2.5: Style Selection

Read [frontend-slides/STYLE_PRESETS.md](frontend-slides/STYLE_PRESETS.md) for all available presets.

Ask ONE question in AskUserQuestion:

**Question — Visual Theme** (header: "Theme"):
Choose a visual style for your autopsy deck.

Options:
- Forensic Dark (recommended) — Dark IDE aesthetic, red/green/amber data accents, monospace metrics
- Bold Signal — High-contrast dark with bold typography and sharp color blocks
- Electric Studio — Vibrant dark theme with electric blue and neon accents
- Browse all presets — Show full list from STYLE_PRESETS.md

If user picks "Browse all presets", list all available presets from STYLE_PRESETS.md and let them choose.

### Applying the Theme

1. Read [viewport-base-autopsy.css](viewport-base-autopsy.css) — include its FULL contents in the `<style>` block. This is a fork of frontend-slides' viewport-base.css with scroll-snap removed and sizing tuned for data-heavy content.
2. Read [frontend-slides/animation-patterns.md](frontend-slides/animation-patterns.md) — use for reveal animations.
3. Load the selected preset's CSS variables from STYLE_PRESETS.md.

### If Forensic Dark is selected (default):

Apply these CSS variable overrides ON TOP of the selected preset:

```css
:root {
    --bg-primary: #08090d;
    --bg-card: #12141a;
    --bg-card-hover: #1a1d26;
    --text-primary: #e8e4df;
    --text-secondary: #8a8580;
    --text-muted: #5c5856;
    --accent-primary: #dc2626;
    --accent-primary-dim: #991b1b;
    --accent-primary-glow: rgba(220, 38, 38, 0.15);
    --accent-secondary: #d97706;
    --accent-secondary-glow: rgba(217, 119, 6, 0.15);
    --accent-success: #10b981;
    --accent-success-glow: rgba(16, 185, 129, 0.15);
    --border-subtle: rgba(255, 255, 255, 0.06);
    --border-medium: rgba(255, 255, 255, 0.1);
    --font-display: 'DM Serif Display', serif;
    --font-body: 'IBM Plex Sans', sans-serif;
    --font-mono: 'IBM Plex Mono', monospace;
}
```

### If another preset is selected:

Map the autopsy-specific semantic variables to that preset's palette:
- `--accent-primary` → preset's primary/dominant accent (used for: danger, death, pattern highlights)
- `--accent-secondary` → preset's secondary accent (used for: metrics, funding amounts, amber warnings)
- `--accent-success` → preset's green/positive accent (used for: survivors, counter-strategies)
- `--bg-primary`, `--bg-card` → preset's background tones
- `--text-primary`, `--text-secondary`, `--text-muted` → preset's text hierarchy
- `--font-display`, `--font-body`, `--font-mono` → preset's font stack

The component styles (tombstones, mind maps, case cards, etc.) should ONLY reference these semantic variables, never hardcoded hex colors.

---

## Phase 3: Build Presentation

### Setup

Before generating, read these files:
- [viewport-base-autopsy.css](viewport-base-autopsy.css) — **MANDATORY.** Include its FULL contents in the `<style>` block. (Forked from frontend-slides with scroll-snap removed and sizing tuned.)
- [frontend-slides/html-template.md](frontend-slides/html-template.md) — HTML architecture and JS features reference
- [frontend-slides/animation-patterns.md](frontend-slides/animation-patterns.md) — CSS animation patterns reference

### Scrolling Behavior

Scroll-snap is already removed in `viewport-base-autopsy.css`. Users scroll freely.

In the JS controller:
- Arrow keys / Space / PageDown → `scrollIntoView({ behavior: 'smooth' })`
- Mouse wheel → **do NOT override.** Let the browser handle it naturally.
- Touch swipe → **do NOT override.** Natural scrolling.
- Keep progress bar and nav dots (both update via Intersection Observer).

### Pattern Legend (for reference)

| Code | Pattern | Tagline |
|------|---------|---------|
| P1 | Platform Dependency | "You're building a feature platforms will absorb" |
| P2 | Commodity/Open-Source Erosion | "Your moat gets erased by free alternatives" |
| P3 | Timing Mismatch | "The market isn't ready, or it's already gone" |
| P4 | Geographic/Market Ceiling | "Your market is structurally bounded" |
| P5 | Monetization Failure | "Users love it but won't pay" |
| P6 | Feature-Not-Company | "You built a feature, not a business" |
| P7 | Service Disguised as Software | "You raised software money for a service business" |
| P8 | Category Economics Don't Work | "The math fundamentally doesn't add up" |
| P9 | Undercapitalization | "You brought a knife to a gunfight" |
| P10 | Enterprise Sales Cycle vs Runway | "Your buyer takes longer to decide than you have to live" |
| P11 | Serial Pivot Death | "Each pivot resets the clock on a bomb still ticking" |
| P12 | Cold-Start / Network Effects | "You need both sides but can't get either" |
| P13 | Regulatory Risk | "The rules can shut you down overnight" |
| P14 | Founder/Team Fragmentation | "The team broke before the company did" |

---

### Slide Structure (12 slides)

**Every slide MUST fit 100vh with `overflow: hidden`. If content overflows, split.**

**Slide 1 — Graveyard (Visual Hook)**

Title overlaid: "Startup Graveyard: [Industry] × [Business Model]"
Subtitle: "[Tier 1 count] [Industry] companies that didn't make it + [Tier 2 count] others killed by the same patterns · YC 2017-2025"

Use the tombstone CSS from the style reference below. Fog gradient from bottom. Slight random rotations via nth-child.

**TOMBSTONE FILTER RULE:** Only include companies where status = "dead" OR status_detail contains "acqui-hire" / "acquired" with exit_value < $10M or exit_value = null. NEVER include companies with exit_value >= $50M (these are success stories, e.g., SafeBase, Tranch, Sqreen, Clear Genetics, Clever, Optimizely). Companies on the Survivors slide (Slide 7) must NEVER appear as tombstones.

**TOMBSTONE SELECTION (TWO TIERS):**

Tier 1 (MUST INCLUDE — shown first, larger tombstones):
- All dead/acqui-hired companies in the user's EXACT industry
- These are the core graveyard — the user should see their own sector's dead first

Tier 2 (FILL REMAINING SPACE — shown smaller/more faded):
- Dead/acqui-hired companies from OTHER industries that share the user's top 3-4 patterns
- Only include if the company's primary_pattern matches one of the user's top patterns
- Cap Tier 2 at 20 companies maximum to prevent grid bloat

**VISUAL DIFFERENTIATION:**
- Tier 1 tombstones: full opacity, slightly larger size
- Tier 2 tombstones: opacity 0.5, slightly smaller, to visually communicate "these are from other industries but died the same way"

Target total: 25-35 tombstones (not 50-80).

---

**Slide 2 — Your Risk Profile**

This is the key framing slide. Show the user's top 3-4 patterns as a visual map/grid, each with:
- Pattern name + code (e.g., "P1 · Platform Absorption")
- Pattern tagline in italics
- Kill count: "Killed [N] of 89 companies in our database"
- A single company name as preview: "including [Company] ([one_liner])"

Layout: 2×2 grid of pattern cards, or vertical stack. Each card color-coded by severity.

Below the grid: "This deck drills into each pattern with real case studies, specific numbers, and survival strategies."

---

**Slides 3-6 — Pattern Deep Dives (one per pattern)**

Each pattern gets ONE slide with mind-map layout:

**Center node:**
- Pattern code + name (e.g., "P1 · Platform Absorption")
- Tagline in italics
- Kill count badge: "Killed [N] companies"

**Left branches: 2-3 case study cards**
Each card MUST include:
- Company name + batch (e.g., "Neptyne · W23")
- Key metric line in monospace (e.g., "$2.5M raised → Microsoft shipped Python in Excel")
- 1-2 sentence death story from death_mechanism (extract the most vivid part)
- If killer_competitor exists, name it explicitly

**Right branches: 1-2 more case cards + 1 survival box**
- Additional case studies (ideally from a DIFFERENT industry than left side)
- Green survival box at bottom with:
  - "Counter-Strategy" label
  - Specific actionable advice derived from critical_decision_point of the cases shown
  - Must include hard thresholds/numbers where available

**CRITICAL: Case selection across industries.**
If the user selected "AI × B2B SaaS" and the pattern is P1 (Platform Absorption):
- Left side: Parabolic (AI) killed by Intercom Fin, Booth AI (AI) killed by Canva
- Right side: Neptyne (Dev Tools) killed by Microsoft, Tydo (SaaS) killed by Shopify
- This cross-industry evidence makes the pattern feel universal, not anecdotal

**Higher info density requirements:**
- Case studies MUST include specific dollar amounts, user counts, timelines, competitor comparisons
- Reference killer_competitor and killer_competitor_funding where available
- Survival strategies MUST include specific thresholds from critical_decision_point
- Include the "what would have worked" counterfactual

---

**Slide 7 — The Survivors**

Show 2-4 companies from comparisons.similar_that_survived that appear frequently across the selected patterns. Layout: side-by-side survivor cards with green accent.

Each survivor card:
- Company name, exit value or current status (big number)
- What they did differently (3-4 bullet lessons)
- Which death pattern they AVOIDED and how

Prefer survivors the audience will recognize (Figma, Cursor, Stripe, etc.)

---

**Slide 8 — Death Timeline**

Horizontal timeline showing when companies matching the user's patterns died (2017→2025):
- Color-coded dots: red=dead, amber=acquired, green=success
- Company name + year at each point
- Cluster labels where multiple companies died in the same year

---

**Slides 9-10 — Survival Playbook**

5 rules, split across 2 slides (3 + 2). Each rule:

1. **Specific to the selected patterns, not generic startup advice**
2. **Derived directly from case studies shown earlier** — reference them by name
3. **Include hard numbers**: dollar thresholds, timeline requirements, competitive benchmarks
4. **Name what to do AND what not to do**
5. **Anti-example**: "[Company] died because they did the opposite"

Format: Numbered cards (01-05) with rule title, 2-3 sentences of specific content, and red anti-example line.

**Example of HIGH DENSITY (good):**
> **01. Charge from Day One — $5/month minimum**
> Station had 40K WAU at 6hrs/day and Piggy had $250M AUM — both with $0 revenue. At $5/month with just 5% conversion, Station would have had $120K ARR. Revenue is the only proof of value. Even Figma charged $12/editor from launch.
> ❌ Station: 40K WAU × 6hrs/day × $0 = dead. Piggy: $250M AUM × 9 years × $0 = dead.

**Example of LOW DENSITY (bad):**
> **01. Make Sure You Monetize**
> It's important to have a revenue model early on.

---

**Slide 11 — The Autopsy Verdict**

A summary diagnosis card specific to the user's combo:
- "[Industry] × [Business Model] Risk Score: [X] of 14 patterns apply"
- Top 3 immediate actions (one sentence each, with numbers)
- "Companies in your exact category that died: [list names]"
- "Companies in your exact category that survived: [list names]"

---

**Slide 12 — The One Rule**

Single powerful closing statement in large display typography.
Specific to the user's deadliest pattern — not generic.

Examples by pattern:
- P1: "If [Platform] could ship your feature by Tuesday, they will. Build what they can't."
- P2: "Your moat has an 18-month expiration date. Monetize now or die open-source."
- P5: "Users will love your product for free. The survivors are the ones who charge for it."
- P9: "$500K against a competitor with $960M is not a business plan."
- P10: "Your buyer takes 18 months to decide. Do you have 36 months of cash?"
- P12: "An empty network has zero value. Solve one community's problem first."
- P13: "Regulators don't care about your runway. Get licensed before you scale."

Small footer text: "Data from 89 YC startups (2017-2025) · startups.rip · [N] companies matched to your risk profile"

---

## Component Style Reference (Semantic Variables Only)

All component styles use semantic CSS variables — never hardcoded hex colors. The theme selected in Phase 2.5 provides the actual color values.

### Tombstone Grid (Slide 1)
```css
.tombstone {
    background: linear-gradient(180deg, var(--bg-card-hover, #1e1e24) 0%, var(--bg-card, #14141a) 100%);
    border-radius: 40% 40% 4px 4px / 25% 25% 4px 4px;
    border: 1px solid var(--border-subtle);
    text-align: center;
    box-shadow: 0 4px 20px rgba(0,0,0,0.4);
}
.tombstone .t-name { color: var(--text-secondary); }
.tombstone .t-year { color: var(--text-muted); }
```
Tier 1: full opacity, larger padding. Tier 2: opacity 0.5, smaller. Slight rotations via nth-child (-2deg, 1deg, -1deg, 2deg, 0deg). Fog gradient from bottom.

### Mind Map (Pattern Slides)
- Center: pattern node with `border: 2px solid var(--accent-primary)` and `box-shadow: 0 0 30px var(--accent-primary-glow)`
- `.mm-code`, `.mm-kills`: `color: var(--accent-primary)`
- Left/right: case study cards with `border-left: 3px solid var(--accent-primary)`
- `.cc-header`: `color: var(--accent-secondary)`
- `.cc-metric`, `.cc-killer`: `color: var(--accent-primary)`
- Survival box: `border-left: 3px solid var(--accent-success)`, `.sb-label`: `color: var(--accent-success)`

### Risk Cards (Slide 2)
- `.risk-card.severity-high::before`: `background: var(--accent-primary)`
- `.risk-card.severity-med::before`: `background: var(--accent-secondary)`
- `.rc-code`, `.rc-kill`: `color: var(--accent-primary)`

### Survivor Cards (Slide 7)
- `border: 1px solid var(--accent-success-glow)`
- `.sv-name`: `color: var(--accent-success)`
- `.sv-avoided`: `color: var(--accent-success)`, `background: var(--accent-success-glow)`

### Timeline Dots (Slide 8)
- `.timeline-dot.dead`: `background: var(--accent-primary)`
- `.timeline-dot.acquired`: `background: var(--accent-secondary)`

### Playbook Rules (Slides 9-10)
- `.rule-num`: `color: var(--accent-primary)`
- `.rule-anti`: `color: var(--accent-primary)`

### Progress Bar & Nav
- Progress bar: `background: linear-gradient(90deg, var(--accent-primary), var(--accent-secondary))`
- `.nav-dot.active`: `background: var(--accent-primary)`, `box-shadow: 0 0 8px var(--accent-primary-glow)`

### Verdict Card (Slide 11)
- `.verdict-score`: `color: var(--accent-primary)`
- `.dead-list h4`: `color: var(--accent-primary)`
- `.alive-list h4`: `color: var(--accent-success)`

### All Slides — Layout & Sizing
- Each slide: 100vh, overflow hidden, no scrolling within slides
- All sizes use clamp() for responsiveness
- Reveal animations triggered by Intersection Observer (.visible class)
- Staggered delays: .s1 through .s15 (0.1s increments)

**Sizing defaults are baked into `viewport-base-autopsy.css`** (larger fonts, wider containers than the generic frontend-slides base). No manual overrides needed.

The overall principle: **content should fill 85-95% of the viewport width and 75-90% of the viewport height**. If the content looks like a small island in a sea of dark background, increase sizes.

---

## Required JS Features

SlidePresentation class with:
- **Keyboard nav**: Arrow keys, Space, PageDown → `scrollIntoView({ behavior: 'smooth' })`
- **NO wheel hijacking**: Browser handles natural scrolling
- **NO touch hijacking**: Browser handles natural swiping
- **Progress bar**: Updates via Intersection Observer (accent-primary → accent-secondary gradient, thin, fixed top)
- **Nav dots**: Right side, updates via Intersection Observer
- Intersection Observer adds `.visible` class at threshold 0.3

---

## Output

1. Save as `[industry]-[model]-autopsy.html` (e.g., `ai-b2b-saas-autopsy.html`)
2. Open with `open [filename].html`
3. Tell user: file location, slide count, top 3 killer patterns, navigation instructions

---

## Data Source

Company data from [startup-database-v2.json](startup-database-v2.json) covering 89 YC startups from batches W17-W25.
Sources: startups.rip, Crunchbase, TechCrunch, founder post-mortems.
