# Tedio v2 — Enhancement Proposal

## Context

This proposal is grounded in four sources:

1. **The existing Tedio web app** — a YouTube history analyzer that generates 6 behavioral insights (Rapid Swipe, Short-Ladder, Late-Night Viewing, Thumbnail Roulette, Single-Channel Reliance, Content Category Balance) with 13 guided quick-action interventions for parents.

2. **Teddy-Xcode (native iOS prototype)** — a separate project that wraps YouTube in a WKWebView and enforces best practices in real-time via injected JavaScript: Shorts removal, channel blocking/whitelisting, Pause & Predict (forces intentional choice on rapid swiping), bedtime wind-down with breathing exercises, session limits with PIN-locked screens, break suggestion banners, and a full Rule Engine (bedtime, daily limit, shorts toggle, break interval, pause-predict toggle).

3. **EU EPRS report on dark patterns (Jan 2025)** — defines dark patterns as practices that "materially distort or impair the ability of recipients to make autonomous and informed choices." The EU is moving toward a Digital Fairness Act with explicit prohibitions. The AI Act specifically prohibits exploiting vulnerabilities based on **age** (Article 5(1)(b)). This gives Tedio a regulatory tailwind.

4. **Parent interviews (10 parents, children ages 1-6)** — key findings:
   - Parents use screens as a "built-in babysitter" during chores, meals, solo parenting (30-45 min/day typical)
   - Biggest fear: long-term neurological/developmental harm, not immediate behavior
   - YouTube algorithm is the #1 villain — auto-redirects to overstimulating knockoff content
   - Parents want **content quality ratings** (stimulation level, educational value, tone), not just age ratings
   - Parents trust **other parents with specific examples** over experts or influencers
   - Simple screen-time limits are NOT what they want — they already manage time; they need help with **what** not **how long**
   - Mom friend group chats are the primary content discovery channel
   - "Boredom is healthy" is a shared value — parents want kids who can self-regulate
   - Guilt is real — a "report card" tone would backfire; parents want to feel empowered, not judged

---

## Design Principles

1. **Tedio does the analysis. Parents do the enforcement — but with exact instructions.** The web app knows WHEN and WHAT the problems are. It generates specific values and walks parents through applying them step-by-step.
2. **Specific over generic.** "Set bedtime to 9:30 PM" because we detected viewing until 10:12 PM — not "consider setting a bedtime."
3. **Allies, not judges.** Frame everything as partnership. No guilt. No report cards.
4. **Build what we can ship from a web app.** No native app dependencies. No features that require infrastructure we can't set up on Railway.

---

## Platform Constraints (What a Web App Can and Cannot Do)

Being honest about what's possible prevents us from shipping half-baked features or misleading parents:

| We CAN do | We CANNOT do |
|-----------|-------------|
| Analyze YouTube history and compute specific values | Deep link into iOS Settings, Screen Time, or any native app |
| Show step-by-step guides with screenshots and personalized values | Auto-configure YouTube, Screen Time, or Family Link for the parent |
| Generate downloadable PDFs (client-side with jsPDF) | Generate signed .mobileconfig profiles (requires Apple MDM) |
| Send emails via SendGrid/Mailgun (requires adding email service) | Send reliable push notifications on iOS (requires PWA install) |
| Classify channels using data already in the upload (watch time, frequency) | Run real-time LLM classification on channel quality (cost, accuracy) |
| Educate parents about techniques like Pause & Predict | Intercept YouTube playback to enforce Pause & Predict (that's native-only) |
| Show a one-time analysis snapshot from uploaded data | Track trends over time (requires periodic re-uploads we can't trigger) |

---

## Proposed Enhancements

### 1. Streamlined Google Takeout Onboarding

**What:** Replace the current "watch this video and go to takeout.google.com" with a guided, step-by-step wizard that stays with the parent throughout the process.

**Why:** The current flow abandons the parent mid-task. Google Takeout is the only realistic data source — the Data Portability API requires security audits, and manual logging is impractical.

**Feasibility:** Fully within reach. This is React UI work on the existing onboarding page. No new backend infrastructure needed.

**Implementation:**
- **Replace the current onboarding page** (`/onboarding`) with a multi-step wizard component:
  - **Step 1/4:** "Let's get your child's YouTube data" — explain what Google Takeout is, what we'll learn from it, set expectations ("even a few weeks of data works")
  - **Step 2/4:** "Open Google Takeout" — direct link to `https://takeout.google.com/settings/takeout/custom/youtube` (pre-filtered to YouTube only) + annotated screenshot showing exactly what to select (YouTube only, JSON format, .zip via email)
  - **Step 3/4:** "Waiting for your file" — "Google will email you a download link in 2-30 minutes. You can close this tab — we'll pick up right where you left off." Save wizard state to localStorage so returning users resume at step 3.
  - **Step 4/4:** "Upload your file" — existing drag-drop uploader, but with explicit path: "Find `Takeout/YouTube and YouTube Music/history/watch-history.json` in your download. That's the only file you need."
- **Progress bar** showing 1/4, 2/4, etc.
- **Persistent state:** If user closes browser and comes back, detect they're mid-onboarding (check localStorage + `first_login` flag) and resume at the right step.
- **Inline help:** Expandable "I'm stuck" sections at each step with troubleshooting (e.g., "I didn't get an email" → "Check spam, or try again — Google sometimes takes up to 30 minutes").

**What we're NOT doing here:** No push notifications or email reminders (that's a separate enhancement requiring email infrastructure). The wizard just saves state so the parent can return.

**Files to change:**
- `Web/ui/src/pages/Onboarding.jsx` — rewrite as multi-step wizard
- `Web/ui/src/css/onboarding.css` — updated styles
- New: `Web/ui/src/components/OnboardingWizard.jsx` — step component
- Add annotated screenshot images to `Web/ui/src/assets/`

---

### 2. Smart Rules Dashboard (Data → Personalized Configuration Guide)

**What:** A new page that transforms Tedio's behavioral insights into specific, data-derived settings values — then walks parents through applying them on their actual devices.

**Why:** The Teddy-Xcode project proved the right model: `bedtime`, `dailyLimitMinutes`, `shortsRemoved`, `breakIntervalMinutes`. Tedio already has the data to compute these values. Instead of "your child watches late at night," generate: "Set bedtime to 9:30 PM on Screen Time." Instead of "consider removing Shorts," say: "Here's how to turn off Shorts in YouTube settings (3 steps)."

**What this is NOT:** This is not "one-tap apply." We cannot auto-configure YouTube or Screen Time from a web app. This is a personalized configuration guide — the values are computed from data, and the setup instructions are step-by-step with screenshots.

**Feasibility:** The data analysis is fully feasible — metrics already exist in `llm.py`. The UI is standard React. The guided wizards are static content with dynamic values injected. PDF generation uses jsPDF (client-side, no server needed).

**Implementation:**

#### Computed Rules (derived from existing metrics in `llm.py`)

| Rule | Data Source (already computed) | How We Calculate | What We Show |
|------|-------------------------------|-----------------|-------------|
| **Bedtime** | Late-Night Viewing timestamps | 30 min before detected late-night viewing start | "Set Screen Time downtime to **9:30 PM**" + step-by-step guide |
| **Daily Limit** | `total_watch_minutes` / days in dataset | Round to nearest 15 min, suggest 20% reduction toward age-appropriate target | "Set daily YouTube limit to **75 minutes**" + guide |
| **Shorts Removal** | Short-Ladder Load score | If >40% Shorts → recommend removal | "Turn off Shorts" + guide for YouTube & YouTube Kids |
| **Break Interval** | Session length data (gap analysis) | 5 min before average continuous session length | "Set YouTube 'Take a Break' reminder to **25 minutes**" + guide |
| **Channel Whitelist** | Top channels by watch time (already extracted) | Sort by minutes, show top 10 | Copyable list of approved channel names for YouTube Kids restricted mode |

**Note on "Blocked Channels":** The original proposal suggested "LLM-flagged overstimulating channels." We're cutting this. We don't have a reliable way to classify channel quality from watch history alone, and an LLM guessing "overstimulating" from a channel name is not responsible. Instead, we surface the full channel list sorted by watch time and let parents decide.

**Note on "Pause & Predict":** The Teddy-Xcode version intercepts YouTube navigation — we can't do that from web. Instead, we'll create an educational card: "What is Pause & Predict?" explaining the technique (pause before a new video, ask 'what do you think this is about?') with practice prompts the parent can use during co-viewing. This is Enhancement #6 territory (Alternative Activities).

#### Guided Setup Instructions (per platform)

Each rule links to a platform-specific, step-by-step guide with:
- Annotated screenshots showing exactly where to tap
- The **specific value computed from their child's data** pre-filled in the instructions
- Platform detection: "Are you using an iPhone or Android?" → show the right guide

Platforms to cover:
- **iOS Screen Time** (Downtime schedule, App Limits) — static screenshots + computed values
- **Google Family Link** (Daily limit, Bedtime) — static screenshots + computed values
- **YouTube app** ("Take a Break" reminder, Restricted Mode) — static screenshots + computed values
- **YouTube Kids** (Approved content only, Timer) — static screenshots + computed values

#### Exportable Documents (client-side PDF generation)

- **Family Media Agreement:** One-page PDF with the child's name and computed rules. "Screen time: 75 min/day. No YouTube after 9:30 PM. Break every 25 minutes. Approved channels: [list]." Generated client-side with jsPDF.
- **Caregiver Cheat Sheet:** Simplified one-pager for nannies/grandparents with the same rules in large, clear text.

**Files to create/change:**
- New: `Web/ui/src/pages/SmartRules.jsx` — main dashboard
- New: `Web/ui/src/components/RuleCard.jsx` — individual rule display
- New: `Web/ui/src/components/SetupWizard.jsx` — step-by-step guide modal
- New: `Web/ui/src/components/PDFExport.jsx` — jsPDF generation
- `Web/backend/app/llm.py` — add rule computation functions (extract bedtime, daily average, session length from existing metrics)
- `Web/backend/app/routes.py` — add `GET /api/rules` endpoint returning computed rules
- Add screenshot assets to `Web/ui/src/assets/guides/`

---

### 3. Dark Pattern Detection Layer

**What:** For each behavioral insight Tedio already generates, add a companion "Why this happens" section naming the specific YouTube design pattern that causes it, with EU regulation references.

**Why:** Parents said "I don't understand why my kid acts this way." Naming the design pattern shifts the narrative from "your kid has a problem" to "YouTube is designed to cause this." The EU agrees — these are prohibited practices under DSA Article 25 and AI Act Article 5.

**Feasibility:** Fully within reach. This is adding static educational content to existing insight detail pages. No new API calls, no new infrastructure. Just content + UI.

**Implementation:**

Each insight detail page (`/insight/:id`) gets a new "Why this happens" expandable section:

| Insight | Dark Pattern Name | Plain-Language Explanation | EU Reference |
|---------|------------------|--------------------------|-------------|
| Rapid Swipe | Autoplay & Infinite Scroll | YouTube automatically plays the next video with no pause or stopping point. Your child doesn't choose to keep watching — the app chooses for them. | DSA Article 25 — prohibits manipulative interface design |
| Short-Ladder | Algorithmic Rabbit Holes | YouTube's recommendation engine is optimized to maximize watch time, not your child's wellbeing. It chains short, stimulating clips to keep them scrolling. | AI Act Article 5(1)(b) — prohibits exploiting age-based vulnerability |
| Late-Night | No Default Time Boundaries | YouTube has no built-in bedtime. Unlike a TV show that ends, YouTube never stops. The absence of boundaries is itself a design choice. | EU Digital Fairness fitness check — platforms should implement protective defaults |
| Thumbnail Roulette | Clickbait Optimization | Thumbnails are A/B tested to maximize clicks, not to accurately represent content. Your child clicks based on the most exciting-looking image, not informed choice. | UCPD Annex I — misleading commercial practices |
| Single-Channel | Filter Bubble | The algorithm reinforces what your child already watches, making it harder to discover diverse content. It optimizes for predictable engagement, not exploration. | DSA Recital 67 — impairing autonomous choice |
| Content Category | Engagement-Maximizing Recommendations | YouTube's algorithm favors content that keeps children watching longest — which tends to be stimulating, not educational. | AI Act — high-risk AI affecting minors' development |

Each card includes:
- What YouTube **does** (the design pattern)
- What YouTube **should** do (the fair alternative)
- What **you** can do about it (link to the relevant Smart Rule)

**Files to change:**
- `Web/ui/src/pages/InsightDetail.jsx` — add "Why this happens" section
- New: `Web/ui/src/data/darkPatterns.js` — static data mapping insight names to dark pattern content
- `Web/ui/src/css/insight-detail.css` — styles for the new section

---

### 4. Enhanced Family Profile

**What:** Expand the user profile beyond child name/age to capture context that makes insights and recommendations more relevant.

**Why:** The current app hardcodes timezone to America/Chicago. Parents want recommendations tied to their values, not generic advice. The profile data feeds into Smart Rules (bedtime, timezone) and framing ("because you value creativity...").

**Feasibility:** Straightforward — form fields stored in MongoDB, used to personalize the UI. Auto-detect timezone from browser.

**Implementation:**
- **New profile fields:**
  - `timezone` — auto-detected from `Intl.DateTimeFormat().resolvedOptions().timeZone`, editable
  - `bedtime` — child's current bedtime (used by Smart Rules to calibrate recommendations)
  - `screen_time_moments` — checkboxes: morning routine, after school, car rides, solo parenting, meals
  - `parent_values` — pick top 3: creativity, boredom tolerance, self-regulation, outdoor play, education, family time
- **Timezone fix:** Pass timezone to `llm.py` preprocessing instead of hardcoding America/Chicago
- **Value-based framing:** Insight cards add a line: "Because you value **creativity**, here's why the Short-Ladder pattern works against that goal."

**Files to change:**
- `Web/ui/src/pages/Settings.jsx` — add new profile fields
- `Web/backend/app/routes.py` — update `PUT /api/settings` to accept new fields
- `Web/backend/app/models/user.py` — add fields to user model
- `Web/backend/app/llm.py` — accept timezone parameter in `_preprocess()`
- `Web/ui/src/pages/InsightDetail.jsx` — add value-based framing

---

### 5. Alternative Activity Engine ("What Instead?")

**What:** Context-aware offline activity suggestions that appear alongside insights and quick actions.

**Why:** Parents said their biggest challenge is having something ready when they take the screen away. Current quick actions say "suggest doing something together" — too vague. Parents need "here's a specific 20-minute activity your 3-year-old can do independently while you cook."

**Feasibility:** Fully within reach. This is a curated JSON dataset with filtering logic. No external API, no LLM call. We write the activity database ourselves.

**Implementation:**
- **Activity database:** ~50 curated activities stored as a JSON file in the frontend, each with:
  - `title`, `description`
  - `age_range` — [min, max] in years
  - `duration_minutes` — how long it takes
  - `time_of_day` — morning / afternoon / evening / bedtime
  - `solo` — boolean (can child do this independently?)
  - `materials` — list of needed items (most should be "none")
  - `transition_tip` — how to smoothly switch from screen to this activity
  - `replaces` — which insight pattern this counters (e.g., "short-ladder" → calm focused activity)
- **"What should we do instead?" button** on every insight card → opens a filtered activity drawer
- **Filtering logic:** Match by child age (from profile), time of day (from browser clock), duration (matched to typical session length from data), solo vs. together
- **Random rotation:** Don't show the same activity twice in a row

**Files to create/change:**
- New: `Web/ui/src/data/activities.json` — curated activity database
- New: `Web/ui/src/components/ActivityDrawer.jsx` — slide-up panel with filtered activities
- `Web/ui/src/pages/InsightDetail.jsx` — add "What Instead?" button
- `Web/ui/src/pages/Summary.jsx` — add activity suggestion to each insight card

---

### 6. EU Dark Pattern Awareness Hub

**What:** A standalone educational page teaching parents about dark patterns in plain language, grounded in EU regulation.

**Why:** Parents want to feel empowered, not helpless. Knowing regulators agree that platforms are exploiting children transforms the narrative from "I'm a bad parent" to "the system is rigged and I can fight back."

**Feasibility:** Fully within reach. Static content page. No API calls, no infrastructure.

**Implementation:**
- **New `/dark-patterns` page** with sections:
  - "What are dark patterns?" — plain-language definition with YouTube-specific examples
  - "What the EU says" — DSA Article 25, AI Act Article 5, Digital Fairness Act overview
  - "Dark patterns in your child's feed" — links to each of the 6 insight types with their dark pattern explanation
  - "What you can do" — link to Smart Rules dashboard
  - "How to report" — links to EU regulatory authorities for filing complaints
- **Glossary section:** Autoplay, infinite scroll, variable reward schedule, engagement optimization, filter bubble — defined for parents, not lawyers
- **Link from every insight card:** "Learn why YouTube does this →"

**Files to create/change:**
- New: `Web/ui/src/pages/DarkPatterns.jsx` — educational hub page
- New: `Web/ui/src/css/dark-patterns.css` — styles
- New: `Web/ui/src/data/darkPatternContent.js` — structured content data
- `Web/ui/src/App.jsx` — add route

---

## What We're Deferring (and Why)

### Scheduled Nudges (Email/Push Notifications)
**Why deferred:** Requires adding email service infrastructure (SendGrid/Mailgun account, email templates, a scheduler/cron on Railway). Web push notifications are unreliable on iOS (requires PWA install to home screen). This is buildable but it's an infrastructure project, not a UI project. Build it after the core features prove value.

### Progress Tracking / Trend Charts
**Why deferred:** The current app processes a one-time upload. Showing "late-night viewing dropped 40% this month" requires the parent to re-upload Takeout data periodically. Google Takeout doesn't support automatic export. Until we have a mechanism to encourage periodic re-uploads (or the Data Portability API becomes accessible), trend data is fiction.

### Content Quality Scoring
**Why deferred:** Highest effort, cold start problem. Needs either LLM classification of every video (expensive, unreliable) or a community of parents rating content (no user base yet). Revisit when user base exists.

### Parent Community Features
**Why deferred:** Needs critical mass. No users = no community.

### Teddy-Xcode Integration
**Why deferred:** The native iOS app is a separate prototype, not deployed or maintained. Building integration points to vaporware wastes effort. If/when the native app ships, add a sync mechanism then.

---

## Priority Order (Build Sequence)

| Priority | Enhancement | Effort | Impact | Rationale |
|----------|------------|--------|--------|-----------|
| **1** | **Streamlined Takeout Onboarding** | Low | Critical | No users without this. Current flow kills adoption. Pure frontend work. |
| **2** | **Enhanced Family Profile** | Low | High | Fixes the hardcoded timezone. Enables personalized framing. Quick backend + frontend work. |
| **3** | **Dark Pattern Detection Layer** | Low | High | Static content added to existing insight pages. Differentiates from every competitor. Highest narrative impact per line of code. |
| **4** | **Smart Rules Dashboard** | Medium | Critical | Core value proposition upgrade. Requires new backend endpoint for rule computation + new frontend page + PDF generation + screenshot assets. |
| **5** | **Alternative Activity Engine** | Low-Medium | Medium | Curated JSON dataset + filtering UI. Concrete value on every insight. |
| **6** | **EU Awareness Hub** | Low | Medium | Static educational page. Brand-positioning, low effort. |

**Note on sequencing:** Enhanced Family Profile (#2) comes before Smart Rules (#4) because the timezone fix and bedtime input feed directly into rule computation. Dark Pattern Detection (#3) comes before Smart Rules because it's lower effort and can ship independently while Smart Rules is being built.

---

## What We're NOT Doing

- **Not auto-configuring devices.** We compute the values and walk parents through applying them. We can't deep link into iOS Settings, Screen Time, or Family Link from a web browser.
- **Not building parental controls.** Tedio is an analysis and guidance tool, not an enforcement tool.
- **Not replacing Google Takeout.** It's the only realistic data source. We're making the process less painful.
- **Not guessing channel quality.** We show watch time data and let parents decide. An LLM guessing "overstimulating" from a channel name is not responsible.
- **Not promising trend tracking without a re-upload mechanism.** One-time analysis gives a snapshot, not a trajectory.
- **Not shaming parents.** Every piece of copy makes parents feel like allies, not defendants.
- **Not building features that need infrastructure we don't have.** Email notifications, push notifications, and real-time sync are deferred until the core experience proves valuable.
