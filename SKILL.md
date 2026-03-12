---
name: startup-autopsy
description: Generate a visual HTML presentation diagnosing startup death risks. Two modes — (A) Input your industry + business model for cross-industry failure pattern analysis, or (B) Name a specific dead startup for a deep company autopsy. Both produce stunning graveyard-themed decks with real data from 89 YC startups (2017-2025). Trigger on industry risk analysis, startup graveyard, failure patterns, company autopsy, or "how do startups in X die".
---

# Startup Autopsy

Two modes of operation:
- **Mode A (Industry):** User inputs industry + business model → cross-industry failure pattern analysis
- **Mode B (Company):** User names a specific startup (or provides a startups.rip URL) → deep autopsy of that one company

---

## Phase 0: Mode Detection

Check the user's message for:
- **A specific company name or startups.rip URL** → Mode B (Company Autopsy)
- **An industry / business model description** → Mode A (Industry Autopsy)
- **Neither** → Ask Phase 1 questions (which include the option to pick Mode B via the "Specific Company" industry option)

**Key rule:** Mode B triggers when the user explicitly provides a company name/URL in their message, OR when the user selects "Specific Company" in Phase 1. The default path is always Mode A (ask questions).

---

## Phase 1: Two-Layer Selection (Mode A default)

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
- Specific Company — I want to autopsy a specific startup (provide name or URL)

**Question 2 — Business Model** (header: "Business Model"):
How does (or will) the product make money?

Options:
- B2C App — Consumer subscription, freemium, consumer-facing app
- B2B SaaS — Selling to businesses on recurring subscription
- API / Dev Platform — Usage-based pricing, developer-facing infrastructure
- Marketplace — Commission/take-rate on transactions between parties

If the user already specified both in their message, skip and go directly to Phase 2.

**If the user selects "Specific Company":** Skip Question 2 (Business Model). Instead, ask a single follow-up AskUserQuestion:

**Question — Company Name** (header: "Company"):
Which startup do you want to autopsy? Enter the company name or a startups.rip URL.

Options:
- Browse startups.rip — I'll look for a company on startups.rip
- I have a name — I'll type the company name (use "Other" to type it)

Once the user provides a company name or URL, proceed to Phase 1B (Company Research).

---

## Phase 1B: Company Research (Mode B only)

When the user provides a specific company name or startups.rip URL:

### Step 1: Gather Data

1. **If startups.rip URL provided:** Fetch the page with WebFetch and extract all available data
2. **If company name only:** Fetch `https://startups.rip/company/[name]` (lowercase, hyphenated). If 404, use WebSearch to find the company's post-mortem data
3. **Check the local database:** Read [startup-database-v2.json](startup-database-v2.json) and search for the company. If found, merge database data with web-fetched data

### Step 2: Build Company Profile

Extract or research these fields:
- **Basics:** Name, founded date, shutdown/acquisition date, location
- **Founders:** Names, notable backgrounds (especially if well-known, e.g., Garry Tan)
- **Product:** What it did, key innovation, target users
- **Funding:** All rounds with amounts, lead investors, total raised
- **Traction:** Peak metrics (users, revenue, traffic, growth rates)
- **Death metrics:** Metrics at time of death/acquisition (to show the decline)
- **Competitors:** Who won and why, with specific numbers
- **Death story:** What went wrong — the narrative arc from rise to fall
- **Key decisions:** The 2-3 moments where things could have gone differently

### Step 3: Map to Failure Patterns

Assign 3-4 patterns from the Pattern Legend that best explain this company's death. For each pattern:
- Write a 1-sentence explanation of how it applies to THIS company specifically
- Identify the key evidence (metrics, timeline, competitor actions)
- Find 2-3 other companies from the database that died of the same pattern (for cross-reference)

### Step 4: Identify Survivors

Find 2-3 well-known companies that were in the same space but survived. For each:
- What they did differently
- Their current valuation/status
- Which pattern they avoided

Then proceed to Phase 2.5 (Style Selection) → Phase 3B (Build Company Presentation).

---

### Phase 3B: Company Autopsy Slide Structure (Mode B — 12 slides)

**Every slide MUST fit 100vh with `overflow: hidden`.**

Same setup as Phase 3 (read viewport-base-autopsy.css, html-template.md, animation-patterns.md). Same theme system. Same JS features.

**Slide 1 — Company Tombstone (Visual Hook)**

A single large tombstone center-screen, not a grid. The company's name and dates displayed prominently.

```
        ┌─────────────┐
        │     RIP      │
        │              │
        │  [Company]   │
        │  2008–2013   │
        │              │
        │ $10.14M raised│
        │  $0 revenue  │
        └──────────────┘
```

Background: same graveyard atmosphere (fog, particles, radial glow). Subtitle below tombstone: "[one-line description of what the company did]"

**Slide 2 — Company Profile (The Patient)**

2×2 grid introducing the company:
- **Top-left card:** Product — what it did, key innovation, target users
- **Top-right card:** Team — founders, notable backgrounds, team size
- **Bottom-left card:** Funding — rounds, investors, total raised (monospace numbers)
- **Bottom-right card:** Peak Traction — best metrics (users, growth rate, traffic)

This slide answers: "What was this company and why did people care?"

**Slide 3 — The Rise (Timeline)**

Visual timeline from founding to peak, showing key milestones:
- Founding → YC batch → First traction milestone → Funding rounds → Peak metric
- Each milestone as a node on a horizontal or diagonal timeline
- Green accent for growth moments
- The timeline should feel like "things were going well..."

**Slide 4 — What Killed [Company] (Pattern Grid)**

2×2 grid of the 3-4 failure patterns, same layout as Mode A's Slide 2. Each card:
- Pattern code + name
- Pattern tagline in italics
- 1-sentence explanation specific to THIS company
- Key evidence line in monospace

Title: "Four Patterns That Killed [Company]"

**Slides 5-7 — Pattern Deep Dives (mind map, 3 slides for top 3 patterns)**

Same mind-map layout as Mode A, BUT the center node and one branch are about the TARGET COMPANY. The other branches show cross-references from the database.

**Center node:** Pattern name + how it applies to [Company]
**Left branch:** [Company]'s specific story for this pattern (with detailed metrics, timeline, key quotes)
**Right branches:** 1-2 other companies from the database that died of the same pattern
**Bottom:** Counter-strategy / what would have worked

This structure says: "[Company] died of Pattern X — and so did these others."

**Slide 8 — The Decline (Metrics Visualization)**

Show the key metric decline that tells the death story:
- Peak number (big, green) → Death number (big, red)
- Percentage decline prominently displayed
- Timeline context (how long the decline took)
- What happened during the decline (key events annotated)

Example for Posterous: "15M monthly visitors → 1.33M · 91% decline in 12 months"

**Slide 9 — What Survived**

Same as Mode A's Slide 7. Show 2-3 companies that were in the same space but survived.
Each card: name, current valuation, what they did differently, which patterns they avoided.

**Slide 10 — Lessons (Survival Playbook)**

4-5 lessons specific to this company's death, each:
- Lesson title
- What [Company] did wrong (specific)
- What they should have done instead (specific, with numbers)
- Anti-example referencing the company

**Slide 11 — The Autopsy Verdict**

Summary card:
- "[Company] Risk Score: [X] of 14 patterns applied"
- Cause of death: 1-sentence summary
- "If [Company] had done [X], they could have been [Survivor]"
- Key counterfactual with specific numbers

**Slide 12 — The One Rule**

Single powerful closing statement specific to this company's deadliest pattern.
Same format as Mode A.

Footer: "Data from startups.rip + 89 YC startups (2017-2025)"

### Output (Mode B)

1. Save as `[company-name]-autopsy.html` (e.g., `posterous-autopsy.html`)
2. Open with `open [filename].html`
3. Tell user: file location, slide count, top 3 failure patterns, navigation instructions

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

### Design Intelligence (from ui-ux-pro-max)

When making visual decisions, consult these data files from [ui-ux-data/](ui-ux-data/):
- [ui-ux-data/styles.csv](ui-ux-data/styles.csv) — 50 design styles with CSS implementation details. Use to select appropriate visual style beyond the built-in presets (e.g., Dark Mode OLED row #7 for forensic dark, or pick a contrasting style for non-default themes).
- [ui-ux-data/typography.csv](ui-ux-data/typography.csv) — 50 font pairings with Google Fonts URLs. Use when the user picks a non-default theme or requests different typography. Row #9 (Developer Mono: JetBrains Mono + IBM Plex Sans) is the closest match to Forensic Dark's default.
- [ui-ux-data/colors.csv](ui-ux-data/colors.csv) — 21 color palettes by product type. Cross-reference when mapping themes to semantic autopsy variables (accent-primary, accent-secondary, accent-success).
- [ui-ux-data/charts.csv](ui-ux-data/charts.csv) — Chart type guidance. Reference for Slide 8 (timeline/decline visualization) to pick the best chart format for the data being shown.
- [ui-ux-data/ux-guidelines.csv](ui-ux-data/ux-guidelines.csv) — UX best practices for layout, spacing, and interaction patterns.

**When to read these files:** Only when generating the presentation (Phase 3/3B). Do NOT read all files upfront — pick the 1-2 most relevant based on the user's theme choice and content needs.

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

**Must look like an actual timeline, not a text list.** A horizontal track with visible dots, connecting line, and company cards dropping below.

**Structure:**
```
    2018      2019       2020       2021       2022       2023       2024       2025
──────●─────────●──●───────●──────────●──●───────●──●──────●─────────●●●●●●──────
      │         │  │       │          │  │       │  │      │         ││││││
   [card]    [card][card] [card]   [card][card] ...       ...      [cards]
```

**CSS for the timeline track:**
```css
.timeline-track {
    position: relative;
    width: 100%;
    height: 4px;
    background: var(--border-medium);
    border-radius: 2px;
    margin: clamp(1rem, 2vh, 2rem) 0;
}
/* Gradient overlay on track */
.timeline-track::after {
    content: '';
    position: absolute; inset: 0;
    background: linear-gradient(90deg, var(--accent-primary-dim), var(--accent-primary), var(--accent-secondary));
    border-radius: 2px;
}
```

**Year markers sit ON the track:**
```css
.timeline-year {
    position: absolute;
    top: 50%;
    transform: translateX(-50%) translateY(-50%);
}
.timeline-year .year-dot {
    width: 12px; height: 12px;
    border-radius: 50%;
    background: var(--bg-primary);
    border: 2px solid var(--accent-secondary);
    margin: 0 auto clamp(4px, 0.5vh, 8px);
}
.timeline-year .year-label {
    font-family: var(--font-mono);
    font-size: clamp(0.7rem, 0.9vw, 0.85rem);
    color: var(--accent-secondary);
    font-weight: 600;
    text-align: center;
    position: absolute;
    top: -24px; left: 50%;
    transform: translateX(-50%);
    white-space: nowrap;
}
```

**Company entries drop BELOW the track as mini-cards:**
```css
.timeline-entry {
    position: absolute;
    top: 20px;  /* below the track */
    transform: translateX(-50%);
    text-align: center;
    max-width: clamp(80px, 10vw, 120px);
}
.timeline-entry .entry-dot {
    width: 8px; height: 8px;
    border-radius: 50%;
    margin: 0 auto 4px;
}
.timeline-entry .entry-dot.dead { background: var(--accent-primary); box-shadow: 0 0 6px var(--accent-primary-glow); }
.timeline-entry .entry-dot.acquired { background: var(--accent-secondary); box-shadow: 0 0 6px var(--accent-secondary-glow); }
.timeline-entry .entry-name {
    font-size: clamp(0.6rem, 0.8vw, 0.75rem);
    color: var(--text-secondary);
    line-height: 1.3;
}
/* Vertical connector line from track to entry */
.timeline-entry::before {
    content: '';
    position: absolute;
    top: -20px; left: 50%;
    width: 1px; height: 16px;
    background: var(--border-medium);
}
```

**Layout approach:** Use a container with `position: relative` for the track. Year markers are positioned with percentage-based `left` values (e.g., 2018=0%, 2025=100%). Companies within each year are stacked vertically below their year marker with small offsets to avoid overlap.

**For years with many companies (2024-2025):** Stack entries in 2 columns below the year dot, or use smaller font. Add a "peak extinction" badge: `"Peak: 2024-2025 (N companies)"` in monospace with red accent.

**Legend:** Below the timeline container, show colored dots with labels: `● Dead` (red) `● Acquired` (amber). Keep it small and left-aligned.

**Animation:** The track line draws from left to right (via `width` transition from 0 to 100%), then dots appear with staggered delays.

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

**The graveyard must look like an actual graveyard, not an admin dashboard.** Tombstones need visible contrast against the background, texture, and atmosphere.

**Grid layout:** Tombstones fill the ENTIRE viewport using `position: absolute; inset: 0`. Title overlays on top with gradient fade. Fog overlays from bottom.

```css
/* Grid fills entire slide */
.tombstone-grid {
    position: absolute; inset: 0;
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(clamp(90px, 10vw, 140px), 1fr));
    gap: clamp(10px, 1.5vw, 18px);
    padding: clamp(2rem, 4vw, 4rem) clamp(2rem, 5vw, 5rem);
    align-content: center;
    z-index: 1;
}

/* Each tombstone = gravestone shape with VISIBLE contrast */
.tombstone {
    background: linear-gradient(180deg, #2a2a32 0%, #1a1a22 60%, #141418 100%);
    border-radius: 40% 40% 4px 4px / 25% 25% 4px 4px;
    border: 1px solid rgba(255, 255, 255, 0.08);
    border-top: 1px solid rgba(255, 255, 255, 0.12);
    box-shadow: 0 4px 20px rgba(0,0,0,0.5), inset 0 1px 0 rgba(255,255,255,0.05);
    aspect-ratio: 3 / 4;
    display: flex; flex-direction: column;
    align-items: center; justify-content: center;
    text-align: center;
    padding: clamp(6px, 1vw, 12px);
    opacity: 0;
    animation: tombstone-rise 0.8s ease forwards;
}

/* Stone texture via pseudo-element */
.tombstone::before {
    content: '';
    position: absolute; inset: 0;
    border-radius: inherit;
    background: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)' opacity='0.03'/%3E%3C/svg%3E");
    pointer-events: none;
}

/* RIP engraving */
.tombstone .rip {
    font-family: var(--font-mono);
    font-size: clamp(0.4rem, 0.6vw, 0.5rem);
    color: rgba(255, 255, 255, 0.25);
    letter-spacing: 0.2em;
    text-transform: uppercase;
    margin-bottom: 3px;
}
.tombstone .name {
    font-family: var(--font-display);
    font-size: clamp(0.65rem, 1vw, 0.9rem);
    color: var(--text-primary);
    line-height: 1.2;
    text-shadow: 0 1px 3px rgba(0,0,0,0.5);
}
.tombstone .batch {
    font-family: var(--font-mono);
    font-size: clamp(0.45rem, 0.7vw, 0.6rem);
    color: var(--text-muted);
    margin-top: 2px;
}
```

**Rotations via nth-child:**
```css
.tombstone:nth-child(5n+1) { --rot: -2deg; }
.tombstone:nth-child(5n+2) { --rot: 1deg; }
.tombstone:nth-child(5n+3) { --rot: -1deg; }
.tombstone:nth-child(5n+4) { --rot: 2deg; }
.tombstone:nth-child(5n+5) { --rot: 0deg; }

@keyframes tombstone-rise {
    from { opacity: 0; transform: translateY(20px) rotate(var(--rot, 0deg)); }
    to { opacity: 1; transform: translateY(0) rotate(var(--rot, 0deg)); }
}
```

**Tier differentiation:**
- Tier 1: animation ends at `opacity: 1`, normal size
- Tier 2: separate keyframes ending at `opacity: 0.35` and `scale(0.85)`

**Staggered animation:** Each tombstone gets `animation-delay` incrementing by 0.02s via inline style.

**Atmosphere layers (z-index stack):**
1. `.tombstone-grid` → z-index: 1
2. `.graveyard-fog` (bottom 35%, gradient to transparent) → z-index: 5
3. `.particle` (red dots floating upward from bottom: 0) → z-index: 6
4. `.graveyard-header` (title + subtitle, gradient fade from top) → z-index: 10

**Header overlay:** `background: linear-gradient(180deg, rgba(8,9,13,0.95) 0%, rgba(8,9,13,0.7) 60%, transparent 100%)`

**Red radial glow:** `.graveyard-slide { background: radial-gradient(ellipse at 50% 20%, rgba(220,38,38,0.06), transparent 50%), var(--bg-primary); }`

**Particles:** 5-7 tiny 2px dots with `animation: float-up linear infinite` rising from bottom to `-100vh`. Durations 8-12s, staggered delays.

**Target tombstone count:** 40-55 total to fill the viewport grid. Add extra Tier 2 companies from the database if needed.

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

### Timeline (Slide 8)
- `.timeline-track`: 4px horizontal line with gradient (accent-primary → accent-secondary), full width
- `.timeline-track::after`: gradient overlay drawing animation (left to right)
- `.timeline-year`: positioned on track with percentage `left`, contains `.year-dot` (12px circle on track) + `.year-label` (above track)
- `.timeline-entry`: drops below track with `::before` vertical connector line (1px), contains `.entry-dot` + `.entry-name`
- `.entry-dot.dead`: `background: var(--accent-primary)`, `box-shadow: 0 0 6px var(--accent-primary-glow)`
- `.entry-dot.acquired`: `background: var(--accent-secondary)`, `box-shadow: 0 0 6px var(--accent-secondary-glow)`
- Legend below: colored dots with labels, small monospace text

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

## Output (Mode A)

1. Save as `[industry]-[model]-autopsy.html` (e.g., `ai-b2b-saas-autopsy.html`)
2. Open with `open [filename].html`
3. Tell user: file location, slide count, top 3 killer patterns, navigation instructions

---

## Data Source

Company data from [startup-database-v2.json](startup-database-v2.json) covering 89 YC startups from batches W17-W25.
Sources: startups.rip, Crunchbase, TechCrunch, founder post-mortems.
