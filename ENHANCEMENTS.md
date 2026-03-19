# Tedio v2 — Enhancement Proposal

## Context

This proposal is grounded in three sources:

1. **The existing Tedio app** — a YouTube history analyzer that generates 6 behavioral insights (Rapid Swipe, Short-Ladder, Late-Night Viewing, Thumbnail Roulette, Single-Channel Reliance, Content Category Balance) with 13 guided quick-action interventions for parents.

2. **EU EPRS report on dark patterns (Jan 2025)** — defines dark patterns as practices that "materially distort or impair the ability of recipients to make autonomous and informed choices." The EU is moving toward a Digital Fairness Act with explicit prohibitions. The AI Act specifically prohibits exploiting vulnerabilities based on **age**. This gives Tedio a regulatory tailwind — parents need tools to understand how platforms manipulate their children.

3. **Parent interviews (10 parents, children ages 1-6)** — key findings:
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

## Proposed Enhancements

### 1. Dark Pattern Detection Layer

**What:** Go beyond behavioral metrics. Name the specific dark patterns YouTube uses on children and show parents exactly how each one works.

**Why:** The EU defines dark patterns as manipulative choice architecture. Parents in interviews said they don't understand *why* their kid can't stop watching. Tedio currently says "your child has a high Rapid Swipe score" — but doesn't explain that YouTube's autoplay, infinite scroll, and thumbnail optimization are *designed* to cause this.

**Implementation:**
- New insight category: **"Platform Design Triggers"**
- For each existing insight, add a companion card explaining the dark pattern behind it:
  - Rapid Swipe → **Autoplay & Infinite Scroll** (DSA Article 25 violation)
  - Short-Ladder → **Algorithmic Rabbit Holes** (AI Act Article 5(1)(b) — exploiting age vulnerability)
  - Late-Night → **No Default Time Boundaries** (absence of "fairness by design")
  - Thumbnail Roulette → **Clickbait Optimization** (manipulative visual design)
  - Single-Channel → **Filter Bubble / Echo Chamber** (limiting autonomous choice)
  - Content Category → **Engagement-Maximizing Recommendation** (not developmentally appropriate)
- Each dark pattern card includes:
  - Plain-language explanation of the design trick
  - Visual diagram of how it works
  - EU regulation reference (DSA, AI Act, UCPD)
  - What the platform *should* do vs. what it actually does

**Parent interview alignment:** Parents said "I don't understand why my kid acts this way." This gives them the *why* — it's not their kid's fault, it's designed manipulation.

---

### 2. Content Quality Scoring (The "Yelp for Kids' Content")

**What:** Rate individual YouTube channels/shows on dimensions parents actually care about, powered by community reviews + LLM analysis.

**Why:** Interview finding #1: parents don't want screen *time* limits — they want help choosing *quality* content. One parent said: "I want ratings based on quality rather than just G or PG, including measures like how many scenes per minute, stimulation level, and educational value." Another: "What would be helpful is a website that says 'my kid loves classical music... this show uses classical music and has xyz.'"

**Implementation:**
- **Stimulation Score** (1-5): Scene change frequency, volume spikes, visual intensity
  - Derived from YouTube video metadata + LLM analysis of channel patterns
- **Educational Value** (1-5): Does it teach? Phonics, social skills, problem-solving?
- **Tone Rating**: Polite language? Characters model good behavior? (Parent flagged Peppa Pig's whiny tone as a red flag standard ratings miss)
- **Situational Tags**: "Good for winding down", "Calming for bedtime", "OK for a 10-min car ride"
- **Parent Reviews**: Quick emoji-tap reviews (under 30 seconds — parents said they won't write long reviews, but will tap tags)
  - Review prompts: Was this calming or overstimulating? Did your child learn something? Would you let it autoplay?
- **Channel Report Cards**: Aggregate scores per channel based on community + AI analysis

**Parent interview alignment:** One parent asked for exactly this — "a recommendation system that rates content based on quality, stimulation level, and educational value rather than just age." Another said "I'd leave a review if it only took 30 seconds with emoji tags."

---

### 3. Personalized "Digital Wellness Plan" (Replace the Report Card)

**What:** Instead of presenting insights as a judgment, frame them as a collaborative, personalized family wellness plan.

**Why:** Multiple parents flagged guilt as a barrier. One said: "A screen time report card adds to mom guilt." Another: "Make it feel more personal and close to the child." The current app presents data that *could* feel accusatory. A wellness plan reframes the same data as a partnership.

**Implementation:**
- **Family Profile**: Beyond child name/age, capture:
  - Family timezone (fixes hardcoded America/Chicago)
  - Typical screen-time moments (morning routine, after school, car rides, solo parenting)
  - Parent's top values (from interview themes: creativity, boredom tolerance, self-regulation, outdoor play)
  - Content preferences (calm shows, educational, no annoying songs)
- **Weekly Wellness Plan** (not "report"):
  - "This week, try replacing 1 evening Shorts session with a Bluey episode" (specific, non-judgmental)
  - Progress tracking: "You completed 3 of 4 goals this week" (not "your child watched 2 hours too much")
  - Celebrate wins: "Late-night viewing dropped 40% since last month"
- **"Because You Value..."** framing: Tie every recommendation back to the parent's stated values
  - Parent values creativity → "Because you value creativity, here's why the Short-Ladder pattern works against that goal"

**Parent interview alignment:** "Going into parenting knowing who you are and what is more important for you." Parents want tools that respect their values, not impose rules.

---

### 4. Alternative Activity Engine ("What Instead?")

**What:** When Tedio detects a problematic pattern, suggest specific offline alternatives matched to the context.

**Why:** Parents said their biggest challenge is having something ready when they take the screen away. One parent's success: "enrolled in activities which reduced screen hours." The current app has generic tips like "suggest doing something together" — these need to be specific and contextual.

**Implementation:**
- **Context-aware suggestions** based on:
  - Time of day (morning → active play, evening → calm activities, bedtime → reading)
  - Child's age (2yr → sensory play, 5yr → building/crafts, 6yr → board games)
  - Weather/season (pull from weather API)
  - Duration needed (parent said they need 30-40 min — match activity duration)
- **Activity library** with:
  - "Waiting-time kits" (expandable from current feature)
  - Age-appropriate offline activities with materials list
  - "5-minute transition activities" for when you're turning off the screen
  - "Parent needs a break" activities that kids can do independently
- **One-tap "What should we do instead?"** button on every insight card

**Parent interview alignment:** "Need to keep things occupied, best thing is to step out and about." Parents need concrete, ready-to-go alternatives, not abstract advice.

---

### 5. Parent Community Features ("Mom Group Chat, Built In")

**What:** Lightweight community layer where parents share what works for them.

**Why:** Every single interviewed parent cited other parents as their #1 source of content recommendations and parenting strategies. One has a dedicated group chat with 10+ moms. Another said she trusts "a mom with a kid just like yours" over experts. The current app is entirely solo — you upload data and read AI-generated insights alone.

**Implementation:**
- **"What worked for me"** stories attached to each quick action
  - Parents can share a 1-2 sentence tip after completing an action
  - Upvote system so best tips surface
- **Content recommendations feed**: "Parents of 4-year-olds are watching..."
  - Filtered by child age group
  - Shows community-rated channels/shows
- **Anonymous by default** — no social profiles, just parent-to-parent tips
- **Moderated**: LLM-assisted moderation to keep it helpful and judgment-free

**Parent interview alignment:** "Talks to mom friends about good shows to help kids learn. Has a groupchat with them." This formalizes what parents already do informally.

---

### 6. Simplified Onboarding (Kill Google Takeout)

**What:** Replace the manual Google Takeout JSON upload with easier data input methods.

**Why:** The current flow requires parents to go to takeout.google.com, select YouTube, download a zip, extract JSON, and upload. This is a massive friction point that will kill adoption. Most parents in the interviews aren't technical — they're stay-at-home moms, nurses, reporters.

**Implementation:**
- **Option A: Manual diary mode** — Parent logs what their child watched today (show name, ~duration, time of day). Simple form, takes 2 minutes. LLM still generates insights from this simpler data.
- **Option B: Screen recording upload** — Parent takes a screenshot of YouTube's "Watch History" page; OCR + LLM extracts the data.
- **Option C: Keep Takeout but with a guided walkthrough** — In-app step-by-step with screenshots, progress indicator, and "we'll wait while you do this" flow.
- **Option D: Browser extension** (future) — Auto-captures YouTube history with one click.

**Parent interview alignment:** These parents are busy — "content is chosen quickly, often no time to curate." A 15-step data export process won't work.

---

### 7. EU Dark Pattern Awareness Hub

**What:** Educational section that teaches parents about dark patterns in plain language, grounded in EU regulation.

**Why:** The EU is pushing toward a Digital Fairness Act. Parents don't know that the design tricks frustrating them are *illegal* or *being regulated*. This positions Tedio as an advocacy tool, not just a monitoring tool.

**Implementation:**
- **"Know Your Rights"** page:
  - What the DSA says about dark patterns (Article 25)
  - What the AI Act says about exploiting children (Article 5)
  - What "fairness by design" means and why platforms should comply
  - How to report dark patterns to EU authorities
- **Per-insight legal context**: Each insight links to the specific EU regulation it violates
- **"Dark Pattern of the Week"**: Rotating educational content about platform manipulation techniques
  - Autoplay, infinite scroll, notification urgency, variable reward schedules, social proof manipulation

**Parent interview alignment:** Parents want to feel empowered. Knowing the law is on their side transforms the narrative from "I'm failing as a parent" to "platforms are designed to exploit my child."

---

## Priority Order (Recommended Build Sequence)

| Priority | Enhancement | Effort | Impact | Rationale |
|----------|------------|--------|--------|-----------|
| 1 | Simplified Onboarding | Medium | Critical | No users without this — current flow kills adoption |
| 2 | Dark Pattern Detection Layer | Low | High | Extends existing insights, differentiates from competitors |
| 3 | Personalized Wellness Plan | Medium | High | Directly addresses guilt concern, increases retention |
| 4 | Content Quality Scoring | High | High | #1 parent request, but needs community + data infrastructure |
| 5 | Alternative Activity Engine | Medium | Medium | Concrete value-add to every insight |
| 6 | EU Dark Pattern Awareness Hub | Low | Medium | Educational, positions brand, low effort |
| 7 | Parent Community Features | High | Medium | Powerful but needs critical mass to work |

---

## What We're NOT Doing

- **Not building parental controls** — Tedio educates and empowers, it doesn't enforce. Parents said they don't want another restriction tool.
- **Not replacing YouTube** — Parents will keep using it. We help them use it better.
- **Not shaming parents** — Every piece of copy should make parents feel like allies, not defendants.
- **Not building a social network** — Community features are lightweight, anonymous, and focused on tips, not profiles.
