# Watchpoint — Concept Design Document

## Vision

**Watchpoint** is a web-based story-driven learning game that teaches players how to improve at Overwatch through a gamified mission system wrapped inside the narrative journey of a character called **Sleepy** — a total rookie rising through the ranks.

The user doesn't just complete tutorials. They experience Sleepy's story — the frustration, the breakthroughs, the rivalries — and every mission they complete drives both the story forward and their own real-world skill development.

---

## 1. The Story Layer — Sleepy's Journey

### Who is Sleepy?

Sleepy is a fixed protagonist with their own personality, voice, and backstory. The user follows Sleepy's arc — they don't create a custom character. This gives the narrative real weight: Sleepy has opinions, makes mistakes, celebrates wins, and grows in ways that mirror what real players experience.

**Starting point:** Sleepy is a complete rookie who just installed Overwatch. Overwhelmed, clueless, but determined. The opening sequence establishes Sleepy downloading the game, getting destroyed in their first match, and deciding they're going to figure this out.

### How the Story is Told

The story is delivered through three formats woven between missions:

- **Story Panels** — Illustrated dialogue sequences (visual novel style). Short, punchy, character-driven. These carry the emotional beats: Sleepy meeting a mentor, getting trash-talked, finding a team.
- **Mission Briefings** — Every mission opens with a narrative frame. Not just "learn about team composition" but "Sleepy's new teammate tells them they need to stop picking random heroes."
- **Debriefs** — After completing a mission, Sleepy reflects on what happened. This ties the gameplay skill back to the story ("So THAT'S why my team kept dying — we had no shield tank").

### The Narrative Structure — Trunk & Branches

The story has two layers:

**The Trunk (Main Story Arc)**

A linear sequence of major story chapters that every player experiences in order. These are the emotional spine of the game — Sleepy's big moments. Each chapter is unlocked by completing enough campaign branches.

| Chapter | Story Beat | Unlocked By |
|---------|-----------|-------------|
| 1 — "Player One" | Sleepy installs OW, gets destroyed, decides to learn | Start of game |
| 2 — "Finding Your Feet" | Sleepy discovers the three roles, picks a direction | Completing "Boot Camp" campaign |
| 3 — "The Squad" | Sleepy joins a casual team, learns to play with others | Completing 2 of the Chapter 2 campaigns |
| 4 — "The Wall" | Sleepy hits a plateau, gets frustrated, considers quitting | Completing 2 of the Chapter 3 campaigns |
| 5+ — "The Climb" | Sleepy pushes through, enters competitive, faces real challenges | Future content |

**The Branches (Campaigns)**

Each trunk chapter unlocks 1-3 campaigns. Campaigns are self-contained sub-stories (5-12 missions each) that explore a specific skill domain through Sleepy's narrative. They have their own mini story arc with a beginning, middle, and payoff.

**Critical design rule: campaigns overlap.** You don't finish one before starting the next. Mid-way through a campaign, new campaigns unlock. This prevents topic fatigue and mirrors how real learning works — you don't master positioning before ever thinking about team comp.

Example campaign flow from Chapter 1:

```
Chapter 1: "Player One" (trunk)
  |
  +---> "Boot Camp" campaign (unlocked immediately, ~10 missions)
  |       |
  |       +-- Mission 4 completes --> unlocks "The Aim Lab" campaign
  |       +-- Mission 7 completes --> unlocks "Know Your Enemy" campaign
  |
  +---> "The Aim Lab" campaign (mechanics focus, ~6 missions)
  +---> "Know Your Enemy" campaign (hero knowledge, ~8 missions)
  |
  v
Chapter 2: "Finding Your Feet" (unlocks when Boot Camp is complete)
```

---

## 2. The Mission System

### What is a Mission?

A mission is the atomic unit of the game. Every mission:

- Teaches **one specific thing** (not three, not five — one)
- Has a **narrative frame** (why Sleepy is doing this right now in the story)
- Has a **clear completion condition**
- Awards **XP and potentially cosmetic rewards**
- Takes **5-15 minutes** to complete

### Mission Types

Missions pull from four content formats, depending on what skill they teach:

| Type | Format | Example |
|------|--------|---------|
| **Learn** | Watch/read a breakdown with key takeaways | "Watch how high ground works, answer 3 questions" |
| **Quiz** | Interactive knowledge check, scenario-based | "Which hero counters Pharah? Pick from these options" |
| **Challenge** | In-game task, self-reported completion | "Play 3 games focusing only on using cover. Rate yourself after" |
| **Drill** | In-app interactive exercise | "Look at this team comp. Identify the weakness. Pick a counter" |

### Self-Reporting System

Since there's no Overwatch API integration, challenges use an honor-based self-report model:

1. Mission gives a clear in-game objective ("Play 2 matches where you call out enemy ultimates")
2. User plays Overwatch, comes back to app
3. App asks reflection questions ("How many ults did you track? What was hardest?")
4. User marks mission as complete
5. Reflection answers could influence story flavor text (Sleepy's debrief mirrors the user's experience)

---

## 3. The Skill Tree

### Structure

The skill tree is a visual map of all campaigns and their missions. It's not a traditional RPG tree — it's more like a **constellation map** where clusters of stars (missions) form named constellations (campaigns), and lines connect them showing progression paths.

### Branches for Future Content

The tree is organized into skill domains. MVP only fills out the Fundamentals domain, but the full tree is visible (locked/greyed out) to show the user what's coming:

```
FUNDAMENTALS (MVP)
  +-- Boot Camp
  +-- The Aim Lab
  +-- Know Your Enemy

ROLES (Post-MVP)
  +-- Tank Academy
  +-- DPS Training Ground
  +-- Support School

GAME SENSE (Post-MVP)
  +-- Map Mastery
  +-- Ultimate Economy
  +-- Positioning & Angles

TEAM PLAY (Post-MVP)
  +-- Communication
  +-- Comp Building
  +-- Synergy & Combos

MENTAL GAME (Post-MVP)
  +-- Tilt Management
  +-- VOD Review Habits
  +-- Warm-Up Routines
```

The locked branches create aspiration — the user can see the depth ahead of them and feels motivated to reach it.

---

## 4. The Reward System

Three reward layers run simultaneously to create multiple dopamine loops:

### Layer 1 — XP & Levels

- Every mission completed awards XP (scaled by mission difficulty)
- XP fills a rank meter: **Bronze → Silver → Gold → Platinum → Diamond → Master → Grandmaster**
- These mirror Overwatch's own rank names intentionally — it creates a parallel progression fantasy
- Level-ups trigger a big animated celebration screen

### Layer 2 — Cosmetic Unlocks

- **Profile badges** — earned for campaign completion ("Boot Camp Graduate," "Aim Lab Certified")
- **Profile borders** — unlocked at rank milestones, increasingly elaborate
- **Titles** — displayed under the username ("Rookie," "Tactician," "Shot Caller")
- **Profile icons** — small collectible icons tied to specific achievements

These are displayed on the user's Watchpoint profile page and visible on any future leaderboard/social features.

### Layer 3 — Content Unlocks

- Completing missions unlocks the next missions in the campaign
- Completing campaigns unlocks new campaigns and trunk chapters
- The reward IS the story — you want to see what happens to Sleepy next
- Locked content is visible but greyed out, creating pull

### Milestone Rewards

At key moments, all three layers fire together:

- Finish "Boot Camp" → Rank up to Silver + "Boot Camp Graduate" badge + Chapter 2 trunk cutscene + 2 new campaigns unlock

This stacking creates a powerful "moment" that feels significant.

---

## 5. Visual Design

### Aesthetic Direction

**Hybrid: Overwatch aesthetic + game-like UI energy**

- **Color palette:** Overwatch's signature blues and oranges, with dark backgrounds. Accent colors per skill domain (green for Support campaigns, red for DPS, blue for Tank)
- **Typography:** Futuristic/clean sans-serif fonts. Bold headers, clean body text. Inspired by OW's in-game UI typography
- **UI elements:** Glowing progress bars, animated XP counters, particle effects on unlocks, dramatic reveal animations for new campaigns
- **Story panels:** Illustrated in a semi-stylized art direction. Sleepy should feel like they could exist in the OW universe but are clearly a fan creation, not an official hero

### Key Screens

| Screen | Purpose |
|--------|---------|
| **Dashboard** | Home screen. Shows current campaign progress, next mission, Sleepy's current story beat, XP/rank status |
| **Story Map** | The constellation-style skill tree showing all campaigns, progress, and locked content |
| **Mission View** | The active mission screen — briefing, content, completion flow, debrief |
| **Story Panels** | Full-screen visual novel sequences between major story beats |
| **Profile** | User's rank, badges, titles, borders, stats |
| **Reward Screen** | Animated celebration screen shown on level-ups and campaign completions |

---

## 6. MVP Scope — "Chapter 1: Player One"

### What Ships

The MVP is a fully playable Chapter 1 experience with one complete campaign and two teaser campaigns:

**Trunk:** Chapter 1 ("Player One") — opening story sequence introducing Sleepy

**Campaign: "Boot Camp"** (~10 missions)

| # | Mission | Type | Teaches |
|---|---------|------|---------|
| 1 | "What Is This Game?" | Learn | Overwatch overview — objective-based, not just kills |
| 2 | "The Three Roles" | Learn + Quiz | Tank, DPS, Support — what each does and why it matters |
| 3 | "Your First Match" | Challenge | Play one match, observe what happens, report back |
| 4 | "Reading the Kill Feed" | Learn + Drill | Understanding the kill feed and what information it gives you |
| 5 | "Pick a Starter Hero" | Learn + Quiz | Beginner-friendly heroes per role, pick one to focus on |
| 6 | "The Objective Wins Games" | Learn + Challenge | Play 2 matches with 100% objective focus, self-rate |
| 7 | "Why Did I Die?" | Learn + Drill | Death analysis — common reasons new players die |
| 8 | "Team Comp Basics" | Learn + Quiz | Why 2-2-2 exists, what a "comp" is |
| 9 | "Your First Win Condition" | Drill | Identify what your team needs to do to win a given scenario |
| 10 | "Boot Camp Graduation" | Challenge + Story | Play 3 matches applying everything, final Sleepy story beat |

**Teaser campaigns** (unlocked mid-Boot Camp, only first 1-2 missions playable):

- "The Aim Lab" — first mission only, showing mechanics content exists
- "Know Your Enemy" — first mission only, showing hero matchup content exists

### What's Visible But Locked

The full skill tree is visible, showing all future domains (Roles, Game Sense, Team Play, Mental Game) greyed out. This sells the depth of the product.

### What Doesn't Ship in MVP

- No user accounts / authentication (local storage for progress)
- No social features or leaderboards
- No real Overwatch API integration
- No mobile responsiveness (desktop web only)
- No user-generated content
- Only Chapter 1 trunk content + Boot Camp campaign fully built

---

## 7. Tech Stack (Recommended)

| Layer | Technology | Why |
|-------|-----------|-----|
| Framework | **Next.js (React)** | Fast to build, SSR for SEO, great ecosystem |
| Styling | **Tailwind CSS** | Rapid UI development, easy to maintain OW-inspired design system |
| Animations | **Framer Motion** | Smooth, game-like animations for rewards, transitions, and story panels |
| State / Progress | **localStorage** (MVP) | No backend needed for MVP, progress persists locally |
| Future Auth | **Supabase or Firebase** | When accounts are needed post-MVP |
| Hosting | **Vercel** | Zero-config Next.js deployment |

---

## 8. Success Metrics for MVP

How we know it's working:

- **Completion rate:** Do users finish all 10 Boot Camp missions?
- **Return rate:** Do users come back after their first session?
- **Session length:** Are sessions 10-20 minutes (sweet spot)?
- **Story engagement:** Do users read/click through story panels or skip them?
- **Teaser click-through:** Do users try the unlocked teaser missions?

---

## 9. Post-MVP Roadmap (Vision Only)

| Phase | Adds |
|-------|------|
| **v1.1** | User accounts + cloud save, complete Chapter 2 trunk + 2 full campaigns |
| **v1.2** | Role-specific campaigns (Tank/DPS/Support), profile customization |
| **v2.0** | Social features — friends, leaderboards, compare progress |
| **v2.5** | Community missions — user-submitted challenges |
| **v3.0** | Light Overwatch API integration for stat-enriched missions |
| **v3.5** | Mobile responsive design or dedicated mobile app |
