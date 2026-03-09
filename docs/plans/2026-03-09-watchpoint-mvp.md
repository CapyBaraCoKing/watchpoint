# Watchpoint MVP Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build the Watchpoint MVP — a story-driven Overwatch learning game following Sleepy's journey through the "Boot Camp" campaign with full mission system, XP/reward engine, and constellation skill map.

**Architecture:** Next.js 14 App Router with TypeScript. All game state persisted in localStorage via a typed engine layer. Content is static TypeScript data files — no database or API for MVP. Pages are client components that read from the progress engine and content data.

**Tech Stack:** Next.js 14, TypeScript, Tailwind CSS, Framer Motion, localStorage

---

## Dependency Graph

```
Sprint 1 (serial)
  └─► Sprint 2A ─┐
  └─► Sprint 2B ─┼─► Sprint 3A ─┐
  └─► Sprint 2C ─┘               ├─► Sprint 4 (serial)
                  └─► Sprint 3B ─┤
                  └─► Sprint 3C ─┘
```

**Parallel opportunities:**
- Sprint 2A + 2B + 2C run simultaneously (no shared files)
- Sprint 3A + 3B + 3C run simultaneously (read from Sprint 2 outputs, no shared files)

---

## File Structure

```
src/
  app/
    layout.tsx
    page.tsx                    ← landing / onboarding
    dashboard/page.tsx
    map/page.tsx
    mission/[id]/page.tsx
    story/[panelId]/page.tsx
    profile/page.tsx
  components/
    ui/Button.tsx, Card.tsx, Badge.tsx, ProgressBar.tsx, GlowText.tsx
    mission/MissionShell.tsx, LearnMission.tsx, QuizMission.tsx,
            ChallengeMission.tsx, DrillMission.tsx
    map/ConstellationMap.tsx, CampaignNode.tsx, MissionStar.tsx
    story/StoryPanel.tsx
    rewards/RewardScreen.tsx, XPCounter.tsx, RankUpCelebration.tsx
  data/
    campaigns.ts
    missions/bootcamp.ts, aimlab.ts, knowYourEnemy.ts
    story/chapter1.ts
    skillTree.ts
  engine/
    storage.ts, progress.ts, xp.ts, unlocks.ts
  hooks/
    useProgress.ts
  types/
    index.ts
docs/plans/
.claude/launch.json
```

---

# SPRINT 1 — Foundation (Serial, do first)

## Task 1: Scaffold Next.js project

**Files:**
- Create: `package.json`, `next.config.ts`, `tsconfig.json`, `tailwind.config.ts`, `src/app/layout.tsx`, `src/app/page.tsx`

**Step 1: Scaffold**
```bash
cd C:/Users/Shadow/overwatch-learning=gamification
npx create-next-app@latest . --typescript --tailwind --eslint --app --src-dir --import-alias "@/*" --yes
```
Expected: Project files created, `npm run dev` works on port 3000.

**Step 2: Install dependencies**
```bash
npm install framer-motion
npm install -D @types/node
```

**Step 3: Verify dev server starts**
```bash
npm run dev
```
Expected: Server running at http://localhost:3000

**Step 4: Create `.claude/launch.json`**
```json
{
  "version": "0.0.1",
  "configurations": [
    {
      "name": "Watchpoint Dev",
      "runtimeExecutable": "npm",
      "runtimeArgs": ["run", "dev"],
      "port": 3000
    }
  ]
}
```

**Step 5: Commit**
```bash
git add -A
git commit -m "chore: scaffold Next.js 14 with Tailwind and Framer Motion"
```

---

## Task 2: TypeScript types

**Files:**
- Create: `src/types/index.ts`

**Step 1: Write types**
```typescript
// src/types/index.ts

export type MissionType = 'learn' | 'quiz' | 'challenge' | 'drill'
export type Rank = 'bronze' | 'silver' | 'gold' | 'platinum' | 'diamond' | 'master' | 'grandmaster'
export type CampaignDomain = 'fundamentals' | 'roles' | 'game-sense' | 'team-play' | 'mental-game'

export interface QuizQuestion {
  question: string
  options: string[]
  correctIndex: number
  explanation: string
}

export interface DrillScenario {
  prompt: string
  imageUrl?: string
  options: string[]
  correctIndex: number
  explanation: string
}

export interface ChallengeReflection {
  question: string
  placeholder: string
}

export interface LearnSection {
  heading: string
  body: string
  keyTakeaway?: string
}

export interface Mission {
  id: string
  campaignId: string
  order: number
  title: string
  types: MissionType[]
  narrativeFrame: string
  debrief: string
  xpReward: number
  estimatedMinutes: number
  content: {
    learn?: { sections: LearnSection[]; checkQuestions: QuizQuestion[] }
    quiz?: QuizQuestion[]
    challenge?: { objective: string; tips: string[]; reflections: ChallengeReflection[] }
    drill?: DrillScenario[]
  }
  unlocksCampaigns?: string[]
}

export interface Campaign {
  id: string
  title: string
  description: string
  domain: CampaignDomain
  chapterId: string
  order: number
  missionIds: string[]
  completionBadgeId: string
  storyArc: string
}

export interface StoryPanelSlide {
  speaker?: 'sleepy' | 'narrator' | string
  expression?: 'neutral' | 'frustrated' | 'happy' | 'determined' | 'surprised'
  text: string
}

export interface StoryPanel {
  id: string
  chapterId: string
  slides: StoryPanelSlide[]
}

export interface Chapter {
  id: string
  order: number
  title: string
  storyBeat: string
  campaignIds: string[]
}

export interface Cosmetic {
  id: string
  type: 'badge' | 'border' | 'title' | 'icon'
  label: string
  description: string
}

export interface PlayerProgress {
  username: string
  xp: number
  rank: Rank
  completedMissions: string[]
  completedCampaigns: string[]
  unlockedCampaigns: string[]
  viewedStoryPanels: string[]
  earnedCosmetics: string[]
  activeTitle?: string
  activeBorder?: string
  activeIcon?: string
  createdAt: string
}
```

**Step 2: Commit**
```bash
git add src/types/index.ts
git commit -m "feat: add core TypeScript types for game data model"
```

---

## Task 3: Tailwind OW design tokens

**Files:**
- Modify: `tailwind.config.ts`
- Create: `src/app/globals.css`

**Step 1: Extend Tailwind config**
```typescript
// tailwind.config.ts
import type { Config } from 'tailwindcss'

const config: Config = {
  content: ['./src/**/*.{js,ts,jsx,tsx,mdx}'],
  theme: {
    extend: {
      colors: {
        ow: {
          blue:    '#4FC3F7',
          'blue-dark': '#0288D1',
          orange:  '#FF6D00',
          'orange-light': '#FFB74D',
          dark:    '#0A0E1A',
          'dark-2': '#111827',
          'dark-3': '#1F2937',
          glow:    '#4FC3F7',
          locked:  '#374151',
        },
        domain: {
          fundamentals: '#4FC3F7',
          roles:        '#F59E0B',
          'game-sense': '#10B981',
          'team-play':  '#8B5CF6',
          'mental-game':'#EC4899',
        }
      },
      fontFamily: {
        display: ['var(--font-display)', 'sans-serif'],
        body:    ['var(--font-body)', 'sans-serif'],
      },
      boxShadow: {
        glow:        '0 0 20px rgba(79,195,247,0.4)',
        'glow-orange':'0 0 20px rgba(255,109,0,0.4)',
        'glow-sm':   '0 0 8px rgba(79,195,247,0.3)',
      },
      animation: {
        'pulse-glow': 'pulseGlow 2s ease-in-out infinite',
        'xp-fill':    'xpFill 1s ease-out forwards',
        'float':      'float 3s ease-in-out infinite',
      },
      keyframes: {
        pulseGlow: {
          '0%,100%': { boxShadow: '0 0 10px rgba(79,195,247,0.3)' },
          '50%':     { boxShadow: '0 0 30px rgba(79,195,247,0.7)' },
        },
        float: {
          '0%,100%': { transform: 'translateY(0px)' },
          '50%':     { transform: 'translateY(-6px)' },
        },
      },
    },
  },
  plugins: [],
}
export default config
```

**Step 2: Update globals.css**
```css
/* src/app/globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

:root {
  --font-display: 'Rajdhani', sans-serif;
  --font-body: 'Inter', sans-serif;
}

body {
  background-color: #0A0E1A;
  color: #E5E7EB;
}

@layer utilities {
  .text-glow {
    text-shadow: 0 0 10px rgba(79,195,247,0.8);
  }
  .border-glow {
    border-color: #4FC3F7;
    box-shadow: 0 0 8px rgba(79,195,247,0.4);
  }
}
```

**Step 3: Add Google Fonts to layout.tsx**
```typescript
// src/app/layout.tsx
import type { Metadata } from 'next'
import { Inter, Rajdhani } from 'next/font/google'
import './globals.css'

const inter = Inter({ subsets: ['latin'], variable: '--font-body' })
const rajdhani = Rajdhani({
  subsets: ['latin'],
  weight: ['400', '600', '700'],
  variable: '--font-display'
})

export const metadata: Metadata = {
  title: 'Watchpoint — Learn Overwatch',
  description: "Follow Sleepy's journey from rookie to Overwatch player",
}

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body className={`${inter.variable} ${rajdhani.variable} font-body bg-ow-dark`}>
        {children}
      </body>
    </html>
  )
}
```

**Step 4: Commit**
```bash
git add -A
git commit -m "feat: add OW design tokens to Tailwind config and layout fonts"
```

---

# SPRINT 2A — Progress Engine (Parallel)

> Run simultaneously with Sprint 2B and 2C. Touches only `src/engine/` and `src/hooks/`.

## Task 4: Storage layer

**Files:**
- Create: `src/engine/storage.ts`

**Step 1: Write storage abstraction**
```typescript
// src/engine/storage.ts
import type { PlayerProgress } from '@/types'

const STORAGE_KEY = 'watchpoint_progress'

export const DEFAULT_PROGRESS: PlayerProgress = {
  username: 'Player',
  xp: 0,
  rank: 'bronze',
  completedMissions: [],
  completedCampaigns: [],
  unlockedCampaigns: ['bootcamp'],
  viewedStoryPanels: [],
  earnedCosmetics: [],
  activeTitle: 'Rookie',
  createdAt: new Date().toISOString(),
}

export function loadProgress(): PlayerProgress {
  if (typeof window === 'undefined') return DEFAULT_PROGRESS
  try {
    const raw = localStorage.getItem(STORAGE_KEY)
    return raw ? { ...DEFAULT_PROGRESS, ...JSON.parse(raw) } : DEFAULT_PROGRESS
  } catch {
    return DEFAULT_PROGRESS
  }
}

export function saveProgress(progress: PlayerProgress): void {
  if (typeof window === 'undefined') return
  localStorage.setItem(STORAGE_KEY, JSON.stringify(progress))
}

export function clearProgress(): void {
  if (typeof window === 'undefined') return
  localStorage.removeItem(STORAGE_KEY)
}
```

**Step 2: Commit**
```bash
git add src/engine/storage.ts
git commit -m "feat: add localStorage progress storage layer"
```

---

## Task 5: XP and rank engine

**Files:**
- Create: `src/engine/xp.ts`

**Step 1: Write XP engine**
```typescript
// src/engine/xp.ts
import type { Rank } from '@/types'

export const RANK_THRESHOLDS: Record<Rank, number> = {
  bronze:      0,
  silver:      500,
  gold:        1500,
  platinum:    3500,
  diamond:     7000,
  master:      13000,
  grandmaster: 21000,
}

const RANK_ORDER: Rank[] = ['bronze','silver','gold','platinum','diamond','master','grandmaster']

export function getRankFromXP(xp: number): Rank {
  let rank: Rank = 'bronze'
  for (const r of RANK_ORDER) {
    if (xp >= RANK_THRESHOLDS[r]) rank = r
    else break
  }
  return rank
}

export function getXPToNextRank(xp: number): { current: number; required: number; rank: Rank } | null {
  const currentRank = getRankFromXP(xp)
  const nextIndex = RANK_ORDER.indexOf(currentRank) + 1
  if (nextIndex >= RANK_ORDER.length) return null
  const nextRank = RANK_ORDER[nextIndex]
  return {
    current: xp - RANK_THRESHOLDS[currentRank],
    required: RANK_THRESHOLDS[nextRank] - RANK_THRESHOLDS[currentRank],
    rank: nextRank,
  }
}

export function didRankUp(prevXP: number, newXP: number): Rank | null {
  const prev = getRankFromXP(prevXP)
  const next = getRankFromXP(newXP)
  return prev !== next ? next : null
}
```

**Step 2: Commit**
```bash
git add src/engine/xp.ts
git commit -m "feat: add XP calculation and rank threshold engine"
```

---

## Task 6: Unlock engine

**Files:**
- Create: `src/engine/unlocks.ts`

**Step 1: Write unlock engine**
```typescript
// src/engine/unlocks.ts
import type { PlayerProgress, Mission } from '@/types'
import { getRankFromXP } from './xp'

export function applyMissionCompletion(
  progress: PlayerProgress,
  mission: Mission,
): PlayerProgress {
  const alreadyDone = progress.completedMissions.includes(mission.id)
  if (alreadyDone) return progress

  const newXP = progress.xp + mission.xpReward
  const newCompleted = [...progress.completedMissions, mission.id]

  // Unlock campaigns this mission gates
  const newUnlocked = [...progress.unlockedCampaigns]
  if (mission.unlocksCampaigns) {
    for (const campaignId of mission.unlocksCampaigns) {
      if (!newUnlocked.includes(campaignId)) {
        newUnlocked.push(campaignId)
      }
    }
  }

  return {
    ...progress,
    xp: newXP,
    rank: getRankFromXP(newXP),
    completedMissions: newCompleted,
    unlockedCampaigns: newUnlocked,
  }
}

export function applyCampaignCompletion(
  progress: PlayerProgress,
  campaignId: string,
  badgeId: string,
): PlayerProgress {
  if (progress.completedCampaigns.includes(campaignId)) return progress
  return {
    ...progress,
    completedCampaigns: [...progress.completedCampaigns, campaignId],
    earnedCosmetics: [...progress.earnedCosmetics, badgeId],
  }
}

export function isMissionUnlocked(missionId: string, order: number, progress: PlayerProgress): boolean {
  if (order === 0) return true
  // A mission is unlocked if the previous mission in the campaign is complete
  // (This is checked by the campaign page using mission order)
  return progress.completedMissions.includes(missionId)
}
```

**Step 2: Commit**
```bash
git add src/engine/unlocks.ts
git commit -m "feat: add campaign and mission unlock engine"
```

---

## Task 7: useProgress hook

**Files:**
- Create: `src/hooks/useProgress.ts`

**Step 1: Write hook**
```typescript
// src/hooks/useProgress.ts
'use client'
import { useState, useEffect, useCallback } from 'react'
import type { PlayerProgress, Mission } from '@/types'
import { loadProgress, saveProgress } from '@/engine/storage'
import { applyMissionCompletion, applyCampaignCompletion } from '@/engine/unlocks'
import { didRankUp } from '@/engine/xp'

export function useProgress() {
  const [progress, setProgress] = useState<PlayerProgress | null>(null)
  const [rankUpEvent, setRankUpEvent] = useState<string | null>(null)

  useEffect(() => {
    setProgress(loadProgress())
  }, [])

  const completeMission = useCallback((mission: Mission) => {
    setProgress(prev => {
      if (!prev) return prev
      const newProgress = applyMissionCompletion(prev, mission)
      const rankUp = didRankUp(prev.xp, newProgress.xp)
      if (rankUp) setRankUpEvent(rankUp)
      saveProgress(newProgress)
      return newProgress
    })
  }, [])

  const completeCampaign = useCallback((campaignId: string, badgeId: string) => {
    setProgress(prev => {
      if (!prev) return prev
      const newProgress = applyCampaignCompletion(prev, campaignId, badgeId)
      saveProgress(newProgress)
      return newProgress
    })
  }, [])

  const markStoryPanelViewed = useCallback((panelId: string) => {
    setProgress(prev => {
      if (!prev || prev.viewedStoryPanels.includes(panelId)) return prev
      const newProgress = { ...prev, viewedStoryPanels: [...prev.viewedStoryPanels, panelId] }
      saveProgress(newProgress)
      return newProgress
    })
  }, [])

  const clearRankUpEvent = useCallback(() => setRankUpEvent(null), [])

  return { progress, completeMission, completeCampaign, markStoryPanelViewed, rankUpEvent, clearRankUpEvent }
}
```

**Step 2: Commit**
```bash
git add src/hooks/useProgress.ts
git commit -m "feat: add useProgress React hook for game state management"
```

---

# SPRINT 2B — Content Data (Parallel)

> Run simultaneously with Sprint 2A and 2C. Touches only `src/data/`.

## Task 8: Boot Camp mission data

**Files:**
- Create: `src/data/missions/bootcamp.ts`

**Step 1: Write all 10 Boot Camp missions**
```typescript
// src/data/missions/bootcamp.ts
import type { Mission } from '@/types'

export const bootcampMissions: Mission[] = [
  {
    id: 'bc-01',
    campaignId: 'bootcamp',
    order: 0,
    title: 'What Is This Game?',
    types: ['learn'],
    narrativeFrame: "Sleepy's first match ended in 37 seconds. They died 14 times. They have no idea what happened.",
    debrief: "Okay. So it's not about kills. It never was. The payload isn't going to push itself.",
    xpReward: 75,
    estimatedMinutes: 8,
    content: {
      learn: {
        sections: [
          {
            heading: 'Overwatch is an objective game',
            body: "Killing enemies is a tool, not the goal. Every match has an objective — push a payload, capture a point, escort cargo. Teams that focus on the objective win. Teams that chase kills lose.",
            keyTakeaway: 'Your job is to enable your team to win the objective, not to get the most kills.',
          },
          {
            heading: 'How matches are structured',
            body: "Each match is 2 teams of 6. You pick a hero before the match. Each hero has unique abilities and a role: Tank, Damage, or Support. Maps have specific win conditions — capture both checkpoints, push the payload over the finish line, etc.",
            keyTakeaway: 'Read the objective text at the top of your screen. That tells you exactly what to do.',
          },
        ],
        checkQuestions: [
          {
            question: 'What is the primary goal in most Overwatch matches?',
            options: ['Get the most eliminations', 'Complete the map objective', 'Survive the longest', 'Deal the most damage'],
            correctIndex: 1,
            explanation: 'Objectives win games. Eliminations are a means to an end — they help you push the objective safely.',
          },
        ],
      },
    },
  },
  {
    id: 'bc-02',
    campaignId: 'bootcamp',
    order: 1,
    title: 'The Three Roles',
    types: ['learn', 'quiz'],
    narrativeFrame: "A random teammate typed 'we need a tank' and Sleepy picked Genji. They still don't know what a tank is.",
    debrief: "Tank. Damage. Support. Three jobs. One team needs all three. Sleepy has been instalocking damage this whole time.",
    xpReward: 80,
    estimatedMinutes: 10,
    content: {
      learn: {
        sections: [
          {
            heading: 'Tank — The shield, the space-maker',
            body: "Tanks create space for their team. They have high health and abilities that protect or disrupt. Their job is to push into the enemy team and give their teammates room to work. Good tanks control where the fight happens.",
            keyTakeaway: 'Play in front of your team. Your health pool exists so your team can stand behind you.',
          },
          {
            heading: 'Damage — The threat',
            body: "Damage heroes eliminate enemies. They deal high damage but are easier to kill. Their job is to take out high-priority targets: healers, snipers, key abilities. A damage player who only fights the enemy tank is wasting their kit.",
            keyTakeaway: 'Target priority matters more than raw damage numbers.',
          },
          {
            heading: 'Support — The backbone',
            body: "Supports keep the team alive and amplify their effectiveness. They heal, boost, and enable. A good support player isn't just a healbot — they also deal damage, deny space, and make game-changing plays with their ultimates.",
            keyTakeaway: "You can't win a fight if your team is dead. Keep them alive first.",
          },
        ],
        checkQuestions: [
          {
            question: 'Which role is primarily responsible for creating space and protecting the team?',
            options: ['Damage', 'Tank', 'Support', 'All equally'],
            correctIndex: 1,
            explanation: 'Tanks absorb damage and push into the enemy, creating room for the rest of the team.',
          },
        ],
      },
      quiz: [
        {
          question: 'Your team is struggling to push the payload. Which of these is most likely the problem?',
          options: ['Your damage player needs to deal more total damage', 'Your tank is hiding behind the team instead of leading', 'Your support is using abilities too often', 'The enemy team has more players'],
          correctIndex: 1,
          explanation: 'Tanks create space. If no one is pushing forward, the team has nowhere to go.',
        },
        {
          question: "Your support player has 35% healing efficiency but the highest damage on the team. Is this a problem?",
          options: ['Yes — supports should only heal', 'No — supports should contribute damage too', 'Only if teammates are dying', 'Depends on the hero'],
          correctIndex: 2,
          explanation: 'Supports dealing damage is great, but not if it comes at the cost of teammates dying. Balance is key.',
        },
      ],
    },
  },
  {
    id: 'bc-03',
    campaignId: 'bootcamp',
    order: 2,
    title: 'Your First Match',
    types: ['challenge'],
    narrativeFrame: "Sleepy's been reading. Now it's time to play — but this time, with intention.",
    debrief: "That was different. Actually watching what happened instead of just reacting to it. Sleepy died less. Maybe.",
    xpReward: 100,
    estimatedMinutes: 15,
    content: {
      challenge: {
        objective: "Play one full Overwatch match. Don't worry about winning. Focus on one thing: every time you die, pause for 2 seconds and ask yourself 'why did that happen?'",
        tips: [
          'Play Quick Play, not Competitive',
          'Pick a hero from the role your team is missing',
          'After each death, before respawning, glance at the kill feed',
        ],
        reflections: [
          { question: 'What role did you play?', placeholder: 'Tank, Damage, or Support...' },
          { question: "What killed you the most? (e.g., a specific hero, getting flanked, no heals)", placeholder: 'What was the main cause of your deaths?' },
          { question: 'Did your team win? Do you know why?', placeholder: 'Try to describe what your team did or failed to do...' },
        ],
      },
    },
  },
  {
    id: 'bc-04',
    campaignId: 'bootcamp',
    order: 3,
    title: 'Reading the Kill Feed',
    types: ['learn', 'drill'],
    narrativeFrame: "Sleepy keeps getting killed by someone they never see coming. The kill feed has been telling them exactly who — they just haven't been reading it.",
    debrief: "The kill feed is a real-time war report. Now Sleepy reads it every death. It's like having a sixth sense.",
    xpReward: 90,
    estimatedMinutes: 10,
    unlocksCampaigns: ['aimlab'],
    content: {
      learn: {
        sections: [
          {
            heading: 'What the kill feed tells you',
            body: "The kill feed (top right of screen) shows every elimination in real time. It shows: who died, who killed them, and what ability was used. This tells you which enemies are alive, which ultimates are being used, and whether your teammates are dying.",
            keyTakeaway: 'Check the kill feed after every death and every few seconds during a fight.',
          },
        ],
        checkQuestions: [],
      },
      drill: [
        {
          prompt: "The kill feed shows your main healer just died to an enemy Reaper. What should your team do immediately?",
          options: ['Push forward aggressively', 'Fall back and regroup — you just lost your sustain', 'Ignore it and keep fighting', 'Swap heroes'],
          correctIndex: 1,
          explanation: 'Without a healer, your team will die faster than the enemy. Fall back, let support respawn, reset the fight.',
        },
        {
          prompt: "You see in the kill feed: enemy Genji used 'Dragonblade' and killed 3 of your teammates. What do you know?",
          options: ["Genji's ultimate is fully charged and dangerous", "Genji's ultimate is now on cooldown — probably for 2+ minutes", 'Nothing useful', 'Genji will use it again immediately'],
          correctIndex: 1,
          explanation: 'Ultimates take time to charge. After using Dragonblade, Genji is less of an immediate ult threat.',
        },
      ],
    },
  },
  {
    id: 'bc-05',
    campaignId: 'bootcamp',
    order: 4,
    title: 'Pick a Starter Hero',
    types: ['learn', 'quiz'],
    narrativeFrame: "Sleepy has been picking a different hero every match. They're bad at all of them. Time to commit.",
    debrief: "One hero. One role. Learn it properly. Sleepy picks their hero. This is them now.",
    xpReward: 75,
    estimatedMinutes: 8,
    content: {
      learn: {
        sections: [
          {
            heading: 'Why you should one-trick early',
            body: "Overwatch has 30+ heroes. Trying to learn all of them at once means learning none of them well. Pick one hero per role and stick with it until it feels automatic. The mechanics, positioning, and timing that make a hero effective take real repetition to build.",
            keyTakeaway: 'Depth beats breadth when you are learning. Master one, then expand.',
          },
          {
            heading: 'Best beginner heroes by role',
            body: "Tank: Reinhardt (point-and-click shield, clear role). Damage: Soldier: 76 (hitscan, self-heal, simple kit). Support: Lucio (hard to kill, movement-based, survivable). These heroes have kits that are easy to understand and teach fundamentals well.",
            keyTakeaway: 'Pick a beginner hero, not your favourite. You can branch out after fundamentals.',
          },
        ],
        checkQuestions: [
          {
            question: 'Why is playing one hero consistently better for a new player than switching every match?',
            options: ['It makes you less fun to play against', 'It lets you build muscle memory and learn a hero deeply', 'It prevents counter-picks', 'It gives you more XP'],
            correctIndex: 1,
            explanation: 'Mastering one hero builds transferable fundamentals — game sense, positioning, timing — faster than spreading thin.',
          },
        ],
      },
      quiz: [
        {
          question: "You're a new support player. Which hero is easiest to learn and most survivable?",
          options: ['Ana (requires precise aim, hard mechanics)', 'Lucio (movement-based, hard to kill, team buff)', 'Zenyatta (low mobility, needs good positioning)', 'Moira (easy aim but short range)'],
          correctIndex: 1,
          explanation: 'Lucio is forgiving for new players — his movement kit means you can escape danger easily.',
        },
      ],
    },
  },
  {
    id: 'bc-06',
    campaignId: 'bootcamp',
    order: 5,
    title: 'The Objective Wins Games',
    types: ['learn', 'challenge'],
    narrativeFrame: "Sleepy just had a 30-kill match. Their team lost. They need to understand why.",
    debrief: "30 kills. 0 time on objective. Sleepy gets it now. Kills are a tool. The objective is the mission.",
    xpReward: 110,
    estimatedMinutes: 20,
    content: {
      learn: {
        sections: [
          {
            heading: 'Why kills without objective time lose games',
            body: "You can eliminate every enemy multiple times and still lose if you never stand on the point or push the payload. Dead enemies respawn after 10 seconds. Objectives only progress when someone is on them. Every second you spend chasing kills off-objective is a second you could be winning.",
            keyTakeaway: 'Ask yourself: am I on the objective, or enabling someone who is?',
          },
        ],
        checkQuestions: [],
      },
      challenge: {
        objective: "Play 2 Quick Play matches. Your only goal: spend as much time as possible on the objective. Contest every point. Push every payload. Count how many times you had to choose between chasing a kill and staying on objective.",
        tips: [
          'If the payload is moving without you on it, you need to be on it',
          'Contesting the objective even while losing buys your team time to respawn',
          'It is okay to let enemies run past you if the objective needs defending',
        ],
        reflections: [
          { question: 'How many times did you choose objective over a kill chase?', placeholder: 'e.g., 3 times I stopped chasing and went back to payload...' },
          { question: 'Did focusing on objective feel different from your normal playstyle?', placeholder: 'Describe what changed...' },
        ],
      },
    },
  },
  {
    id: 'bc-07',
    campaignId: 'bootcamp',
    order: 6,
    title: 'Why Did I Die?',
    types: ['learn', 'drill'],
    narrativeFrame: "Sleepy just died in 0.3 seconds and has no idea how. It keeps happening. Time to diagnose.",
    debrief: "It was always one of three things. No cover. Too far from heals. Wrong position. Sleepy sees it now.",
    xpReward: 90,
    estimatedMinutes: 10,
    unlocksCampaigns: ['know-your-enemy'],
    content: {
      learn: {
        sections: [
          {
            heading: 'The three most common death causes for new players',
            body: "1) Standing in the open — no cover means easy to hit. 2) Too far from your support — healers have limited range, move out of it and you get no heals. 3) Overextension — pushing too far ahead of your team means you fight 6v1 instead of 6v6.",
            keyTakeaway: 'After every death, ask: was I in cover? Was I near my healer? Was I too far ahead?',
          },
        ],
        checkQuestions: [],
      },
      drill: [
        {
          prompt: "You're playing Reinhardt. You charged into the enemy backline, killed their healer, and then died immediately. Was this a good trade?",
          options: ['Yes — you killed their healer, that is worth dying for', 'Sometimes — depends on whether your team followed up', 'No — your team still had to fight 5v5 without their tank', 'It depends on the score'],
          correctIndex: 1,
          explanation: 'A kill on their support is valuable, but only if your team capitalizes. If you died and your team did not push, you gave up your tank for nothing.',
        },
        {
          prompt: "Your healer keeps dying. You notice they're always standing in the front line next to the tank. What should you tell them?",
          options: ['Nothing, healers should be close to everyone', 'Stay behind the tanks and at mid-range — not in the front', 'Switch to a different hero', 'Play more aggressively'],
          correctIndex: 1,
          explanation: 'Supports are high-value targets. They should stay behind the tank line, healing from protected positions.',
        },
      ],
    },
  },
  {
    id: 'bc-08',
    campaignId: 'bootcamp',
    order: 7,
    title: 'Team Comp Basics',
    types: ['learn', 'quiz'],
    narrativeFrame: "Sleepy's team has 5 damage players. Their healer left. They lost in 90 seconds. This needs addressing.",
    debrief: "2 tanks, 2 damage, 2 supports. It exists for a reason. Sleepy is never going to be the 5th damage player again.",
    xpReward: 80,
    estimatedMinutes: 10,
    content: {
      learn: {
        sections: [
          {
            heading: 'Why 2-2-2 exists',
            body: "Overwatch locks you into role queue for competitive play: 2 tanks, 2 damage, 2 supports. This exists because teams with no heals die fast, teams with no tanks have no space to work, and teams with no damage can not eliminate threats. Each role fills a gap the others cannot.",
            keyTakeaway: 'When you queue, you are signing up to fill a role. Fill it properly.',
          },
          {
            heading: 'What makes a comp fall apart',
            body: "Two damage players both picking snipers means no close-range threat. Two tanks both picking barriers means no mobility. Two supports both picking healers with no utility means no peel. Comps should cover different ranges, engagement styles, and threats.",
            keyTakeaway: 'Look at what your team already has before picking your hero.',
          },
        ],
        checkQuestions: [],
      },
      quiz: [
        {
          question: 'Your team already has a Bastion (stationary, long-range damage) and a Hanzo (sniper). What type of damage hero would balance this comp?',
          options: ['Another sniper for more long-range damage', 'A mobile, close-range flanker like Tracer or Genji', 'A tank hero since you need space', 'A second healer for safety'],
          correctIndex: 1,
          explanation: 'Your team lacks close-range presence. A flanker can dive the enemy backline that long-range heroes cannot reach.',
        },
      ],
    },
  },
  {
    id: 'bc-09',
    campaignId: 'bootcamp',
    order: 8,
    title: 'Your First Win Condition',
    types: ['drill'],
    narrativeFrame: "Sleepy stares at the hero select screen. 5 heroes picked. They need to figure out what this team can actually do.",
    debrief: "Reading a team comp and knowing what it can win feels like a superpower. Sleepy isn't guessing anymore.",
    xpReward: 95,
    estimatedMinutes: 12,
    content: {
      drill: [
        {
          prompt: "Your team: Reinhardt, Zarya, Soldier: 76, Reaper, Ana, Lucio. The enemy team has 3 snipers on high ground. What is your win condition?",
          options: ['Out-snipe them from range', 'Use Reinhardt shield to close distance and fight in close quarters where snipers are weak', 'Switch all heroes to snipers', 'Stall the game until time runs out'],
          correctIndex: 1,
          explanation: 'Your comp is built for close-range brawling. Reinhardt shield covers your advance. Get close and your comp dominates.',
        },
        {
          prompt: "Your team: Wrecking Ball, D.Va, Tracer, Sombra, Zenyatta, Baptiste. What type of playstyle does this comp demand?",
          options: ['Slow and methodical — hold a choke point', 'Dive — fast, mobile, get in and out quickly', 'Poke — deal damage from long range', 'Stall — play for overtime'],
          correctIndex: 1,
          explanation: 'Wrecking Ball, D.Va, and Tracer are all dive heroes. This comp wants to move fast, isolate targets, and disengage.',
        },
        {
          prompt: "You are winning the fight 6v3. The enemy payload is 10 meters from the finish line. What do you do?",
          options: ['Chase the remaining 3 enemies to finish them off', 'Immediately stop the payload — wins are decided by objectives not kills', 'Recall everyone to base', 'Push with 3 players, hold back with 3'],
          correctIndex: 1,
          explanation: 'A 10-meter payload push wins the game. No kill is worth letting the payload reach the finish line.',
        },
      ],
    },
  },
  {
    id: 'bc-10',
    campaignId: 'bootcamp',
    order: 9,
    title: 'Boot Camp Graduation',
    types: ['challenge'],
    narrativeFrame: "Sleepy has learned the fundamentals. Now it's time to prove it — not to anyone else, but to themselves.",
    debrief: "That was Boot Camp. Sleepy is not the same player who installed Overwatch with no idea what a tank was. This is just the beginning.",
    xpReward: 150,
    estimatedMinutes: 30,
    content: {
      challenge: {
        objective: "Play 3 Quick Play matches applying everything from Boot Camp. Focus on: role fulfilment, objective time, reading the kill feed, knowing why you died.",
        tips: [
          'Pick one hero and stick with it for all 3 matches',
          'Check the kill feed at least twice per team fight',
          'After each death, name one reason you died before you respawn',
          'Stay near your healer as a damage player, or near your tanks as a support',
        ],
        reflections: [
          { question: 'Did you play the same role all 3 games? Which role?', placeholder: 'e.g., I played support all 3 games as Lucio...' },
          { question: 'What was the single biggest improvement from your very first match to now?', placeholder: 'Compare your very first game to today...' },
          { question: 'What do you still feel weakest at?', placeholder: 'This helps us know what campaign to tackle next...' },
        ],
      },
    },
  },
]
```

**Step 2: Commit**
```bash
git add src/data/missions/bootcamp.ts
git commit -m "feat: add all 10 Boot Camp mission definitions with full content"
```

---

## Task 9: Campaigns, chapters, skill tree, and teaser data

**Files:**
- Create: `src/data/campaigns.ts`
- Create: `src/data/missions/aimlab.ts`
- Create: `src/data/missions/knowYourEnemy.ts`
- Create: `src/data/story/chapter1.ts`
- Create: `src/data/skillTree.ts`

**Step 1: Write campaigns.ts**
```typescript
// src/data/campaigns.ts
import type { Campaign, Chapter } from '@/types'

export const chapters: Chapter[] = [
  {
    id: 'ch-01',
    order: 0,
    title: 'Player One',
    storyBeat: "Sleepy installs Overwatch, gets destroyed, and decides to figure it out.",
    campaignIds: ['bootcamp'],
  },
  {
    id: 'ch-02',
    order: 1,
    title: 'Finding Your Feet',
    storyBeat: "Sleepy discovers roles, picks a direction, and starts to have a plan.",
    campaignIds: ['aimlab', 'know-your-enemy'],
  },
]

export const campaigns: Campaign[] = [
  {
    id: 'bootcamp',
    title: 'Boot Camp',
    description: "Sleepy's crash course in how Overwatch actually works.",
    domain: 'fundamentals',
    chapterId: 'ch-01',
    order: 0,
    missionIds: ['bc-01','bc-02','bc-03','bc-04','bc-05','bc-06','bc-07','bc-08','bc-09','bc-10'],
    completionBadgeId: 'badge-bootcamp-graduate',
    storyArc: 'From complete rookie to knowing the fundamentals.',
  },
  {
    id: 'aimlab',
    title: 'The Aim Lab',
    description: "Sleepy finds a training partner who will fix their mechanical skills.",
    domain: 'fundamentals',
    chapterId: 'ch-02',
    order: 0,
    missionIds: ['al-01'],
    completionBadgeId: 'badge-aim-lab',
    storyArc: 'Building mechanical foundations.',
  },
  {
    id: 'know-your-enemy',
    title: 'Know Your Enemy',
    description: "Sleepy starts getting hard-countered and decides to learn the roster.",
    domain: 'fundamentals',
    chapterId: 'ch-02',
    order: 1,
    missionIds: ['kye-01'],
    completionBadgeId: 'badge-enemy-expert',
    storyArc: 'Understanding hero matchups.',
  },
]
```

**Step 2: Write teaser missions**
```typescript
// src/data/missions/aimlab.ts
import type { Mission } from '@/types'

export const aimlabMissions: Mission[] = [
  {
    id: 'al-01',
    campaignId: 'aimlab',
    order: 0,
    title: 'Your Aim Baseline',
    types: ['learn'],
    narrativeFrame: "Sleepy keeps missing shots that should hit. A random player tells them about aim training. They decide to try it.",
    debrief: "Every pro player trains their aim. Sleepy just took their first step into mechanics.",
    xpReward: 80,
    estimatedMinutes: 8,
    content: {
      learn: {
        sections: [
          {
            heading: 'Why aim training matters',
            body: "Mechanical skill is the foundation under every other skill. Good game sense means nothing if you can't convert opportunities. This campaign covers crosshair placement, tracking, and flicking — the three types of aim in Overwatch.",
            keyTakeaway: 'Aim is a skill, not a talent. It is trained through deliberate practice.',
          },
        ],
        checkQuestions: [],
      },
    },
  },
]
```

```typescript
// src/data/missions/knowYourEnemy.ts
import type { Mission } from '@/types'

export const knowYourEnemyMissions: Mission[] = [
  {
    id: 'kye-01',
    campaignId: 'know-your-enemy',
    order: 0,
    title: 'What Just Killed Me?',
    types: ['learn'],
    narrativeFrame: "Sleepy died three times to the same hero and still doesn't know what that hero does.",
    debrief: "Knowing what the enemy can do is half the battle. Sleepy is no longer afraid of what they don't understand.",
    xpReward: 80,
    estimatedMinutes: 8,
    content: {
      learn: {
        sections: [
          {
            heading: 'Why you need to know enemy kits',
            body: "You cannot counter what you don't understand. Every hero has clear weaknesses if you know their kit. This campaign walks through the most common threats you'll face and how to deal with them.",
            keyTakeaway: 'When a hero keeps killing you, look them up. 5 minutes of research saves 10 games of frustration.',
          },
        ],
        checkQuestions: [],
      },
    },
  },
]
```

**Step 3: Write Chapter 1 story panels**
```typescript
// src/data/story/chapter1.ts
import type { StoryPanel } from '@/types'

export const chapter1Panels: StoryPanel[] = [
  {
    id: 'panel-ch01-intro',
    chapterId: 'ch-01',
    slides: [
      { speaker: 'narrator', text: 'Somewhere, a computer boots up. A game installs.' },
      { speaker: 'sleepy', expression: 'neutral', text: "Okay. How hard can it be? It's just a shooter." },
      { speaker: 'narrator', text: '14 deaths. 0 objective time. Match duration: 3 minutes.' },
      { speaker: 'sleepy', expression: 'frustrated', text: "...What just happened?" },
      { speaker: 'sleepy', expression: 'determined', text: "Okay. I'm going to figure this out." },
    ],
  },
  {
    id: 'panel-ch01-bootcamp-complete',
    chapterId: 'ch-01',
    slides: [
      { speaker: 'narrator', text: "Three weeks later. Sleepy's stats have changed." },
      { speaker: 'sleepy', expression: 'happy', text: "I know what a tank does now. I know what the kill feed is. I have a main." },
      { speaker: 'sleepy', expression: 'determined', text: "I'm still losing too many games. There's more to learn." },
      { speaker: 'narrator', text: "There always is. But this is no longer a rookie standing in the open. This is a player." },
    ],
  },
]
```

**Step 4: Write skill tree data**
```typescript
// src/data/skillTree.ts
export type SkillDomain = {
  id: string
  label: string
  description: string
  color: string
  mvp: boolean
  campaigns: { id: string; label: string; available: boolean }[]
}

export const skillDomains: SkillDomain[] = [
  {
    id: 'fundamentals',
    label: 'Fundamentals',
    description: 'The foundation of Overwatch knowledge',
    color: '#4FC3F7',
    mvp: true,
    campaigns: [
      { id: 'bootcamp', label: 'Boot Camp', available: true },
      { id: 'aimlab', label: 'The Aim Lab', available: true },
      { id: 'know-your-enemy', label: 'Know Your Enemy', available: true },
    ],
  },
  {
    id: 'roles',
    label: 'Roles',
    description: 'Deep dives into each role',
    color: '#F59E0B',
    mvp: false,
    campaigns: [
      { id: 'tank-academy', label: 'Tank Academy', available: false },
      { id: 'dps-training', label: 'DPS Training Ground', available: false },
      { id: 'support-school', label: 'Support School', available: false },
    ],
  },
  {
    id: 'game-sense',
    label: 'Game Sense',
    description: 'Reading the game at a higher level',
    color: '#10B981',
    mvp: false,
    campaigns: [
      { id: 'map-mastery', label: 'Map Mastery', available: false },
      { id: 'ult-economy', label: 'Ultimate Economy', available: false },
      { id: 'positioning', label: 'Positioning & Angles', available: false },
    ],
  },
  {
    id: 'team-play',
    label: 'Team Play',
    description: 'Winning through coordination',
    color: '#8B5CF6',
    mvp: false,
    campaigns: [
      { id: 'communication', label: 'Communication', available: false },
      { id: 'comp-building', label: 'Comp Building', available: false },
      { id: 'synergy', label: 'Synergy & Combos', available: false },
    ],
  },
  {
    id: 'mental-game',
    label: 'Mental Game',
    description: 'The psychological side of competitive play',
    color: '#EC4899',
    mvp: false,
    campaigns: [
      { id: 'tilt-management', label: 'Tilt Management', available: false },
      { id: 'vod-review', label: 'VOD Review Habits', available: false },
      { id: 'warmup', label: 'Warm-Up Routines', available: false },
    ],
  },
]
```

**Step 5: Create data index**
```typescript
// src/data/index.ts
export { bootcampMissions } from './missions/bootcamp'
export { aimlabMissions } from './missions/aimlab'
export { knowYourEnemyMissions } from './missions/knowYourEnemy'
export { campaigns, chapters } from './campaigns'
export { chapter1Panels } from './story/chapter1'
export { skillDomains } from './skillTree'

import { bootcampMissions } from './missions/bootcamp'
import { aimlabMissions } from './missions/aimlab'
import { knowYourEnemyMissions } from './missions/knowYourEnemy'
import type { Mission } from '@/types'

export const allMissions: Mission[] = [
  ...bootcampMissions,
  ...aimlabMissions,
  ...knowYourEnemyMissions,
]

export function getMissionById(id: string): Mission | undefined {
  return allMissions.find(m => m.id === id)
}
```

**Step 6: Commit**
```bash
git add src/data/
git commit -m "feat: add all game content data — campaigns, missions, story panels, skill tree"
```

---

# SPRINT 2C — UI Component Library (Parallel)

> Run simultaneously with Sprint 2A and 2B. Touches only `src/components/ui/`.

## Task 10: Base UI components

**Files:**
- Create: `src/components/ui/Button.tsx`
- Create: `src/components/ui/Card.tsx`
- Create: `src/components/ui/Badge.tsx`
- Create: `src/components/ui/ProgressBar.tsx`
- Create: `src/components/ui/GlowText.tsx`
- Create: `src/components/ui/index.ts`

**Step 1: Button component**
```typescript
// src/components/ui/Button.tsx
'use client'
import { motion } from 'framer-motion'
import { type ButtonHTMLAttributes } from 'react'

interface ButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'ghost' | 'danger'
  size?: 'sm' | 'md' | 'lg'
  glow?: boolean
}

const variants = {
  primary:   'bg-ow-blue text-ow-dark font-semibold hover:bg-ow-blue/90',
  secondary: 'bg-ow-dark-3 border border-ow-blue/40 text-ow-blue hover:border-ow-blue',
  ghost:     'text-gray-400 hover:text-white hover:bg-white/5',
  danger:    'bg-red-600/20 border border-red-500/40 text-red-400 hover:border-red-400',
}
const sizes = {
  sm: 'px-3 py-1.5 text-sm',
  md: 'px-5 py-2.5 text-base',
  lg: 'px-8 py-3.5 text-lg',
}

export function Button({ variant = 'primary', size = 'md', glow, className = '', children, ...props }: ButtonProps) {
  return (
    <motion.button
      whileHover={{ scale: 1.02 }}
      whileTap={{ scale: 0.98 }}
      className={`
        rounded font-display tracking-wide transition-all duration-150
        ${variants[variant]} ${sizes[size]}
        ${glow ? 'shadow-glow' : ''}
        disabled:opacity-40 disabled:cursor-not-allowed
        ${className}
      `}
      {...(props as any)}
    >
      {children}
    </motion.button>
  )
}
```

**Step 2: Card component**
```typescript
// src/components/ui/Card.tsx
import { type ReactNode } from 'react'

interface CardProps {
  children: ReactNode
  className?: string
  glow?: boolean
  locked?: boolean
}

export function Card({ children, className = '', glow, locked }: CardProps) {
  return (
    <div className={`
      rounded-lg border bg-ow-dark-2 p-4
      ${glow ? 'border-ow-blue/60 shadow-glow-sm' : 'border-white/10'}
      ${locked ? 'opacity-50 grayscale' : ''}
      ${className}
    `}>
      {children}
    </div>
  )
}
```

**Step 3: ProgressBar component**
```typescript
// src/components/ui/ProgressBar.tsx
'use client'
import { motion } from 'framer-motion'

interface ProgressBarProps {
  value: number      // 0-100
  color?: string
  label?: string
  animated?: boolean
  height?: 'sm' | 'md' | 'lg'
}

const heights = { sm: 'h-1.5', md: 'h-2.5', lg: 'h-4' }

export function ProgressBar({ value, color = '#4FC3F7', label, animated = true, height = 'md' }: ProgressBarProps) {
  return (
    <div className="w-full">
      {label && <div className="mb-1 flex justify-between text-xs text-gray-400"><span>{label}</span><span>{Math.round(value)}%</span></div>}
      <div className={`w-full rounded-full bg-white/10 overflow-hidden ${heights[height]}`}>
        <motion.div
          className="h-full rounded-full"
          style={{ backgroundColor: color }}
          initial={animated ? { width: 0 } : { width: `${value}%` }}
          animate={{ width: `${value}%` }}
          transition={{ duration: 1, ease: 'easeOut' }}
        />
      </div>
    </div>
  )
}
```

**Step 4: GlowText and Badge**
```typescript
// src/components/ui/GlowText.tsx
import { type ReactNode } from 'react'

export function GlowText({ children, color = 'ow-blue', className = '' }: {
  children: ReactNode; color?: string; className?: string
}) {
  return (
    <span className={`text-${color} text-glow font-display ${className}`}>
      {children}
    </span>
  )
}
```

```typescript
// src/components/ui/Badge.tsx
interface BadgeProps {
  label: string
  color?: 'blue' | 'orange' | 'green' | 'purple' | 'pink'
  size?: 'sm' | 'md'
}
const colors = {
  blue:   'bg-ow-blue/10 text-ow-blue border-ow-blue/30',
  orange: 'bg-ow-orange/10 text-ow-orange border-ow-orange/30',
  green:  'bg-emerald-500/10 text-emerald-400 border-emerald-500/30',
  purple: 'bg-purple-500/10 text-purple-400 border-purple-500/30',
  pink:   'bg-pink-500/10 text-pink-400 border-pink-500/30',
}
export function Badge({ label, color = 'blue', size = 'md' }: BadgeProps) {
  return (
    <span className={`
      inline-flex items-center rounded border font-display tracking-wider uppercase
      ${colors[color]}
      ${size === 'sm' ? 'px-2 py-0.5 text-xs' : 'px-3 py-1 text-sm'}
    `}>
      {label}
    </span>
  )
}
```

**Step 5: Export barrel**
```typescript
// src/components/ui/index.ts
export { Button } from './Button'
export { Card } from './Card'
export { Badge } from './Badge'
export { ProgressBar } from './ProgressBar'
export { GlowText } from './GlowText'
```

**Step 6: Commit**
```bash
git add src/components/ui/
git commit -m "feat: add OW-styled base UI component library"
```

---

# SPRINT 3A — Dashboard + Story Map (Parallel)

> Depends on Sprint 2A + 2B + 2C. Touches `src/components/map/` and `src/app/dashboard/`, `src/app/map/`.

## Task 11: Constellation Map components

**Files:**
- Create: `src/components/map/ConstellationMap.tsx`
- Create: `src/components/map/CampaignNode.tsx`
- Create: `src/components/map/MissionStar.tsx`

**Step 1: MissionStar**
```typescript
// src/components/map/MissionStar.tsx
'use client'
import { motion } from 'framer-motion'

interface MissionStarProps {
  completed: boolean
  active: boolean
  locked: boolean
  size?: number
  onClick?: () => void
}

export function MissionStar({ completed, active, locked, size = 12, onClick }: MissionStarProps) {
  const color = completed ? '#4FC3F7' : active ? '#FF6D00' : locked ? '#374151' : '#6B7280'
  return (
    <motion.button
      onClick={onClick}
      disabled={locked}
      whileHover={!locked ? { scale: 1.3 } : {}}
      animate={active ? { scale: [1, 1.15, 1] } : {}}
      transition={{ duration: 2, repeat: active ? Infinity : 0 }}
      className="relative flex items-center justify-center rounded-full cursor-pointer disabled:cursor-default"
      style={{ width: size, height: size }}
    >
      <div
        className="rounded-full transition-all duration-300"
        style={{
          width: size,
          height: size,
          backgroundColor: color,
          boxShadow: (completed || active) ? `0 0 ${size}px ${color}80` : 'none',
        }}
      />
    </motion.button>
  )
}
```

**Step 2: CampaignNode**
```typescript
// src/components/map/CampaignNode.tsx
'use client'
import { motion } from 'framer-motion'
import { useRouter } from 'next/navigation'
import type { Campaign } from '@/types'
import type { PlayerProgress } from '@/types'
import { MissionStar } from './MissionStar'
import { allMissions } from '@/data'

interface CampaignNodeProps {
  campaign: Campaign
  progress: PlayerProgress
  x: number
  y: number
}

export function CampaignNode({ campaign, progress, x, y }: CampaignNodeProps) {
  const router = useRouter()
  const isUnlocked = progress.unlockedCampaigns.includes(campaign.id)
  const isComplete = progress.completedCampaigns.includes(campaign.id)
  const missions = allMissions.filter(m => m.campaignId === campaign.id)
  const completedCount = missions.filter(m => progress.completedMissions.includes(m.id)).length

  return (
    <motion.div
      className="absolute"
      style={{ left: x, top: y, transform: 'translate(-50%, -50%)' }}
      initial={{ opacity: 0, scale: 0.8 }}
      animate={{ opacity: 1, scale: 1 }}
      transition={{ duration: 0.4 }}
    >
      <div className={`
        rounded-lg border p-3 w-40 cursor-pointer transition-all duration-200
        ${isUnlocked
          ? 'border-ow-blue/40 bg-ow-dark-2 hover:border-ow-blue shadow-glow-sm'
          : 'border-white/10 bg-ow-dark-2/50 cursor-not-allowed opacity-50'}
      `}
        onClick={() => isUnlocked && router.push(`/campaign/${campaign.id}`)}
      >
        <p className="font-display text-sm font-semibold text-white truncate">{campaign.title}</p>
        <p className="mt-1 text-xs text-gray-400">{completedCount}/{missions.length} missions</p>
        <div className="mt-2 flex gap-1 flex-wrap">
          {missions.map((m, i) => (
            <MissionStar
              key={m.id}
              completed={progress.completedMissions.includes(m.id)}
              active={!progress.completedMissions.includes(m.id) && (i === 0 || progress.completedMissions.includes(missions[i-1]?.id))}
              locked={!isUnlocked}
              size={8}
            />
          ))}
        </div>
        {isComplete && (
          <div className="mt-2 text-xs text-ow-blue font-display">✓ COMPLETE</div>
        )}
        {!isUnlocked && (
          <div className="mt-2 text-xs text-gray-500 font-display">🔒 LOCKED</div>
        )}
      </div>
    </motion.div>
  )
}
```

**Step 3: ConstellationMap page**
```typescript
// src/app/map/page.tsx
'use client'
import { useProgress } from '@/hooks/useProgress'
import { CampaignNode } from '@/components/map/CampaignNode'
import { skillDomains } from '@/data/skillTree'
import { campaigns } from '@/data/campaigns'
import { GlowText } from '@/components/ui'

// Static positions for MVP campaigns
const CAMPAIGN_POSITIONS: Record<string, { x: number; y: number }> = {
  'bootcamp':         { x: 400, y: 200 },
  'aimlab':           { x: 250, y: 380 },
  'know-your-enemy':  { x: 550, y: 380 },
  'tank-academy':     { x: 150, y: 550 },
  'dps-training':     { x: 400, y: 580 },
  'support-school':   { x: 650, y: 550 },
}

export default function MapPage() {
  const { progress } = useProgress()
  if (!progress) return null

  return (
    <div className="min-h-screen bg-ow-dark p-6">
      <div className="mx-auto max-w-4xl">
        <div className="mb-8 text-center">
          <h1 className="font-display text-4xl font-bold"><GlowText>Skill Map</GlowText></h1>
          <p className="mt-2 text-gray-400">Sleepy's journey through the Overwatch learning tree</p>
        </div>

        {/* Domain legend */}
        <div className="mb-6 flex flex-wrap gap-3 justify-center">
          {skillDomains.map(domain => (
            <div key={domain.id} className="flex items-center gap-2 text-sm">
              <div className="h-2 w-2 rounded-full" style={{ backgroundColor: domain.color }} />
              <span className={domain.mvp ? 'text-white' : 'text-gray-500'}>
                {domain.label} {!domain.mvp && '(Coming Soon)'}
              </span>
            </div>
          ))}
        </div>

        {/* Map canvas */}
        <div className="relative mx-auto" style={{ width: 800, height: 700 }}>
          {/* Connection lines (SVG) */}
          <svg className="absolute inset-0 w-full h-full pointer-events-none">
            <line x1="400" y1="200" x2="250" y2="380" stroke="#4FC3F7" strokeWidth="1" strokeOpacity="0.2" strokeDasharray="4 4" />
            <line x1="400" y1="200" x2="550" y2="380" stroke="#4FC3F7" strokeWidth="1" strokeOpacity="0.2" strokeDasharray="4 4" />
          </svg>

          {/* Campaign nodes */}
          {campaigns.map(campaign => {
            const pos = CAMPAIGN_POSITIONS[campaign.id]
            if (!pos) return null
            return (
              <CampaignNode
                key={campaign.id}
                campaign={campaign}
                progress={progress}
                x={pos.x}
                y={pos.y}
              />
            )
          })}
        </div>
      </div>
    </div>
  )
}
```

**Step 4: Dashboard page**
```typescript
// src/app/dashboard/page.tsx
'use client'
import { useRouter } from 'next/navigation'
import { useProgress } from '@/hooks/useProgress'
import { ProgressBar, Button, Card, Badge, GlowText } from '@/components/ui'
import { getXPToNextRank } from '@/engine/xp'
import { allMissions, campaigns } from '@/data'

export default function DashboardPage() {
  const router = useRouter()
  const { progress } = useProgress()
  if (!progress) return null

  const xpInfo = getXPToNextRank(progress.xp)
  const activeCampaigns = campaigns.filter(c => progress.unlockedCampaigns.includes(c.id))

  // Find next incomplete mission across unlocked campaigns
  const nextMission = allMissions.find(m =>
    progress.unlockedCampaigns.includes(m.campaignId) &&
    !progress.completedMissions.includes(m.id)
  )

  return (
    <div className="min-h-screen bg-ow-dark p-6">
      <div className="mx-auto max-w-3xl space-y-6">

        {/* Header */}
        <div className="flex items-center justify-between">
          <div>
            <h1 className="font-display text-3xl font-bold">
              Welcome back, <GlowText>{progress.username}</GlowText>
            </h1>
            <div className="mt-1 flex items-center gap-2">
              <Badge label={progress.rank.toUpperCase()} color="blue" size="sm" />
              {progress.activeTitle && <span className="text-sm text-gray-400">"{progress.activeTitle}"</span>}
            </div>
          </div>
          <Button variant="secondary" onClick={() => router.push('/map')}>Skill Map</Button>
        </div>

        {/* XP Progress */}
        <Card>
          <div className="flex justify-between mb-2">
            <span className="font-display text-sm text-gray-400">XP</span>
            <span className="font-display text-sm text-ow-blue">{progress.xp.toLocaleString()} XP</span>
          </div>
          {xpInfo ? (
            <ProgressBar
              value={(xpInfo.current / xpInfo.required) * 100}
              label={`→ ${xpInfo.rank.toUpperCase()}`}
              height="md"
            />
          ) : (
            <div className="text-center text-ow-orange font-display">GRANDMASTER</div>
          )}
        </Card>

        {/* Next Mission CTA */}
        {nextMission && (
          <Card glow>
            <div className="flex items-center justify-between">
              <div>
                <p className="text-xs uppercase tracking-widest text-ow-blue font-display mb-1">Continue</p>
                <h2 className="font-display text-xl font-bold">{nextMission.title}</h2>
                <p className="text-sm text-gray-400 mt-1 italic">"{nextMission.narrativeFrame}"</p>
                <div className="mt-2 flex gap-2">
                  {nextMission.types.map(t => <Badge key={t} label={t} size="sm" />)}
                  <span className="text-xs text-gray-500 self-center">~{nextMission.estimatedMinutes} min</span>
                </div>
              </div>
              <Button glow onClick={() => router.push(`/mission/${nextMission.id}`)}>
                Start →
              </Button>
            </div>
          </Card>
        )}

        {/* Active Campaigns */}
        <div>
          <h2 className="font-display text-lg font-semibold text-gray-300 mb-3">Active Campaigns</h2>
          <div className="grid gap-3">
            {activeCampaigns.map(campaign => {
              const missions = allMissions.filter(m => m.campaignId === campaign.id)
              const done = missions.filter(m => progress.completedMissions.includes(m.id)).length
              return (
                <Card key={campaign.id}>
                  <div className="flex items-center justify-between">
                    <div>
                      <p className="font-display font-semibold">{campaign.title}</p>
                      <p className="text-xs text-gray-400">{done}/{missions.length} missions complete</p>
                    </div>
                    <ProgressBar
                      value={(done / missions.length) * 100}
                      height="sm"
                      animated={false}
                    />
                  </div>
                </Card>
              )
            })}
          </div>
        </div>

      </div>
    </div>
  )
}
```

**Step 5: Commit**
```bash
git add src/components/map/ src/app/dashboard/ src/app/map/
git commit -m "feat: add Dashboard and Constellation Map pages"
```

---

# SPRINT 3B — Mission View System (Parallel)

> Depends on Sprint 2A + 2B + 2C. Touches `src/components/mission/` and `src/app/mission/`.

## Task 12: Mission shell and all four mission type components

**Files:**
- Create: `src/components/mission/MissionShell.tsx`
- Create: `src/components/mission/LearnMission.tsx`
- Create: `src/components/mission/QuizMission.tsx`
- Create: `src/components/mission/ChallengeMission.tsx`
- Create: `src/components/mission/DrillMission.tsx`
- Create: `src/app/mission/[id]/page.tsx`

**Step 1: MissionShell**
```typescript
// src/components/mission/MissionShell.tsx
'use client'
import { useState } from 'react'
import { motion, AnimatePresence } from 'framer-motion'
import type { Mission } from '@/types'
import { Button, Card, Badge, GlowText } from '@/components/ui'
import { LearnMission } from './LearnMission'
import { QuizMission } from './QuizMission'
import { ChallengeMission } from './ChallengeMission'
import { DrillMission } from './DrillMission'

type Phase = 'briefing' | 'content' | 'debrief' | 'complete'

interface MissionShellProps {
  mission: Mission
  onComplete: () => void
}

export function MissionShell({ mission, onComplete }: MissionShellProps) {
  const [phase, setPhase] = useState<Phase>('briefing')
  const [contentIndex, setContentIndex] = useState(0)

  const contentTypes = mission.types

  const handleContentComplete = () => {
    if (contentIndex < contentTypes.length - 1) {
      setContentIndex(i => i + 1)
    } else {
      setPhase('debrief')
    }
  }

  const renderContent = () => {
    const type = contentTypes[contentIndex]
    if (type === 'learn' && mission.content.learn)
      return <LearnMission content={mission.content.learn} onComplete={handleContentComplete} />
    if (type === 'quiz' && mission.content.quiz)
      return <QuizMission questions={mission.content.quiz} onComplete={handleContentComplete} />
    if (type === 'challenge' && mission.content.challenge)
      return <ChallengeMission challenge={mission.content.challenge} onComplete={handleContentComplete} />
    if (type === 'drill' && mission.content.drill)
      return <DrillMission scenarios={mission.content.drill} onComplete={handleContentComplete} />
    return null
  }

  return (
    <div className="min-h-screen bg-ow-dark p-6">
      <div className="mx-auto max-w-2xl">
        <AnimatePresence mode="wait">

          {phase === 'briefing' && (
            <motion.div key="briefing" initial={{ opacity: 0, y: 20 }} animate={{ opacity: 1, y: 0 }} exit={{ opacity: 0, y: -20 }}>
              <Card glow className="space-y-4">
                <div className="flex gap-2">
                  {mission.types.map(t => <Badge key={t} label={t} />)}
                  <span className="text-xs text-gray-500 self-center ml-auto">~{mission.estimatedMinutes} min · {mission.xpReward} XP</span>
                </div>
                <h1 className="font-display text-3xl font-bold"><GlowText>{mission.title}</GlowText></h1>
                <blockquote className="border-l-2 border-ow-orange pl-4 italic text-gray-300">
                  "{mission.narrativeFrame}"
                </blockquote>
                <Button glow size="lg" className="w-full" onClick={() => setPhase('content')}>
                  Begin Mission →
                </Button>
              </Card>
            </motion.div>
          )}

          {phase === 'content' && (
            <motion.div key={`content-${contentIndex}`} initial={{ opacity: 0, x: 30 }} animate={{ opacity: 1, x: 0 }} exit={{ opacity: 0, x: -30 }}>
              {renderContent()}
            </motion.div>
          )}

          {phase === 'debrief' && (
            <motion.div key="debrief" initial={{ opacity: 0, y: 20 }} animate={{ opacity: 1, y: 0 }} exit={{ opacity: 0 }}>
              <Card className="space-y-4 text-center">
                <div className="text-4xl">✓</div>
                <h2 className="font-display text-2xl font-bold text-ow-blue">Mission Complete</h2>
                <blockquote className="italic text-gray-300 border-l-2 border-ow-blue/40 pl-4 text-left">
                  <span className="text-xs text-ow-blue font-display">SLEEPY // DEBRIEF</span><br />
                  "{mission.debrief}"
                </blockquote>
                <div className="text-ow-orange font-display text-xl">+{mission.xpReward} XP</div>
                <Button glow size="lg" className="w-full" onClick={() => { setPhase('complete'); onComplete() }}>
                  Claim Reward →
                </Button>
              </Card>
            </motion.div>
          )}

        </AnimatePresence>
      </div>
    </div>
  )
}
```

**Step 2: LearnMission**
```typescript
// src/components/mission/LearnMission.tsx
'use client'
import { useState } from 'react'
import type { Mission } from '@/types'
import { Button, Card } from '@/components/ui'

interface LearnMissionProps {
  content: NonNullable<Mission['content']['learn']>
  onComplete: () => void
}

export function LearnMission({ content, onComplete }: LearnMissionProps) {
  const [sectionIndex, setSectionIndex] = useState(0)
  const [quizIndex, setQuizIndex] = useState(0)
  const [phase, setPhase] = useState<'reading' | 'quiz'>('reading')
  const [selected, setSelected] = useState<number | null>(null)

  const section = content.sections[sectionIndex]
  const question = content.checkQuestions[quizIndex]
  const isLast = sectionIndex === content.sections.length - 1

  const nextSection = () => {
    if (!isLast) { setSectionIndex(i => i + 1); return }
    if (content.checkQuestions.length > 0) { setPhase('quiz'); return }
    onComplete()
  }

  const submitAnswer = () => {
    if (quizIndex < content.checkQuestions.length - 1) {
      setQuizIndex(i => i + 1); setSelected(null)
    } else {
      onComplete()
    }
  }

  if (phase === 'quiz' && question) {
    return (
      <Card className="space-y-4">
        <p className="text-xs text-ow-blue font-display uppercase">Knowledge Check</p>
        <p className="font-display text-lg">{question.question}</p>
        <div className="space-y-2">
          {question.options.map((opt, i) => (
            <button key={i} onClick={() => setSelected(i)}
              className={`w-full text-left p-3 rounded border transition-all ${
                selected === i
                  ? i === question.correctIndex ? 'border-green-500 bg-green-500/10 text-green-300' : 'border-red-500 bg-red-500/10 text-red-300'
                  : 'border-white/10 hover:border-ow-blue/40'
              }`}>
              {opt}
            </button>
          ))}
        </div>
        {selected !== null && (
          <div className="space-y-3">
            <p className="text-sm text-gray-400">{question.explanation}</p>
            <Button onClick={submitAnswer} className="w-full">Continue →</Button>
          </div>
        )}
      </Card>
    )
  }

  return (
    <Card className="space-y-4">
      <p className="text-xs text-gray-500 font-display">Section {sectionIndex + 1} of {content.sections.length}</p>
      <h2 className="font-display text-xl font-bold text-white">{section.heading}</h2>
      <p className="text-gray-300 leading-relaxed">{section.body}</p>
      {section.keyTakeaway && (
        <div className="border border-ow-orange/30 bg-ow-orange/5 p-3 rounded">
          <p className="text-xs text-ow-orange font-display uppercase mb-1">Key Takeaway</p>
          <p className="text-sm text-ow-orange/90">{section.keyTakeaway}</p>
        </div>
      )}
      <Button onClick={nextSection} className="w-full">
        {isLast && content.checkQuestions.length === 0 ? 'Complete →' : 'Next →'}
      </Button>
    </Card>
  )
}
```

**Step 3: QuizMission**
```typescript
// src/components/mission/QuizMission.tsx
'use client'
import { useState } from 'react'
import type { QuizQuestion } from '@/types'
import { Button, Card } from '@/components/ui'

interface QuizMissionProps {
  questions: QuizQuestion[]
  onComplete: () => void
}

export function QuizMission({ questions, onComplete }: QuizMissionProps) {
  const [index, setIndex] = useState(0)
  const [selected, setSelected] = useState<number | null>(null)
  const question = questions[index]

  const next = () => {
    if (index < questions.length - 1) { setIndex(i => i + 1); setSelected(null) }
    else onComplete()
  }

  return (
    <Card className="space-y-4">
      <p className="text-xs text-ow-blue font-display uppercase">Question {index + 1} of {questions.length}</p>
      <p className="font-display text-lg">{question.question}</p>
      <div className="space-y-2">
        {question.options.map((opt, i) => (
          <button key={i} onClick={() => !selected && setSelected(i)}
            className={`w-full text-left p-3 rounded border transition-all ${
              selected !== null
                ? i === question.correctIndex ? 'border-green-500 bg-green-500/10 text-green-300'
                  : selected === i ? 'border-red-500 bg-red-500/10 text-red-300' : 'border-white/10 opacity-50'
                : 'border-white/10 hover:border-ow-blue/40'
            }`}>
            {opt}
          </button>
        ))}
      </div>
      {selected !== null && (
        <div className="space-y-3">
          <p className="text-sm text-gray-400">{question.explanation}</p>
          <Button onClick={next} className="w-full">{index === questions.length - 1 ? 'Complete →' : 'Next →'}</Button>
        </div>
      )}
    </Card>
  )
}
```

**Step 4: ChallengeMission**
```typescript
// src/components/mission/ChallengeMission.tsx
'use client'
import { useState } from 'react'
import type { Mission } from '@/types'
import { Button, Card } from '@/components/ui'

interface ChallengeMissionProps {
  challenge: NonNullable<Mission['content']['challenge']>
  onComplete: () => void
}

export function ChallengeMission({ challenge, onComplete }: ChallengeMissionProps) {
  const [phase, setPhase] = useState<'brief' | 'reflection'>('brief')
  const [answers, setAnswers] = useState<string[]>(challenge.reflections.map(() => ''))

  return (
    <Card className="space-y-4">
      {phase === 'brief' ? (
        <>
          <p className="text-xs text-ow-orange font-display uppercase">In-Game Challenge</p>
          <p className="text-gray-300 leading-relaxed">{challenge.objective}</p>
          <ul className="space-y-1">
            {challenge.tips.map((tip, i) => (
              <li key={i} className="text-sm text-gray-400 flex gap-2"><span className="text-ow-blue">›</span>{tip}</li>
            ))}
          </ul>
          <p className="text-xs text-gray-500 italic">Go play Overwatch now, then return here to complete the reflection.</p>
          <Button glow onClick={() => setPhase('reflection')} className="w-full">I've Completed the Challenge →</Button>
        </>
      ) : (
        <>
          <p className="text-xs text-ow-blue font-display uppercase">Mission Reflection</p>
          {challenge.reflections.map((r, i) => (
            <div key={i} className="space-y-1">
              <label className="text-sm text-gray-300">{r.question}</label>
              <textarea
                value={answers[i]}
                onChange={e => { const a = [...answers]; a[i] = e.target.value; setAnswers(a) }}
                placeholder={r.placeholder}
                className="w-full rounded border border-white/10 bg-ow-dark-3 p-2 text-sm text-gray-300 placeholder-gray-600 resize-none focus:border-ow-blue/40 focus:outline-none"
                rows={2}
              />
            </div>
          ))}
          <Button onClick={onComplete} className="w-full" disabled={answers.some(a => a.trim() === '')}>
            Complete Mission →
          </Button>
        </>
      )}
    </Card>
  )
}
```

**Step 5: DrillMission**
```typescript
// src/components/mission/DrillMission.tsx
'use client'
import { useState } from 'react'
import type { DrillScenario } from '@/types'
import { Button, Card } from '@/components/ui'

interface DrillMissionProps {
  scenarios: DrillScenario[]
  onComplete: () => void
}

export function DrillMission({ scenarios, onComplete }: DrillMissionProps) {
  const [index, setIndex] = useState(0)
  const [selected, setSelected] = useState<number | null>(null)
  const scenario = scenarios[index]

  const next = () => {
    if (index < scenarios.length - 1) { setIndex(i => i + 1); setSelected(null) }
    else onComplete()
  }

  return (
    <Card className="space-y-4">
      <p className="text-xs text-ow-blue font-display uppercase">Scenario {index + 1} of {scenarios.length}</p>
      <p className="font-display text-lg">{scenario.prompt}</p>
      <div className="space-y-2">
        {scenario.options.map((opt, i) => (
          <button key={i} onClick={() => !selected && setSelected(i)}
            className={`w-full text-left p-3 rounded border transition-all text-sm ${
              selected !== null
                ? i === scenario.correctIndex ? 'border-green-500 bg-green-500/10 text-green-300'
                  : selected === i ? 'border-red-500 bg-red-500/10 text-red-300' : 'border-white/10 opacity-50'
                : 'border-white/10 hover:border-ow-blue/40 hover:bg-ow-blue/5'
            }`}>
            {opt}
          </button>
        ))}
      </div>
      {selected !== null && (
        <div className="space-y-3">
          <p className="text-sm text-gray-400 border-l-2 border-ow-blue/40 pl-3">{scenario.explanation}</p>
          <Button onClick={next} className="w-full">{index === scenarios.length - 1 ? 'Complete →' : 'Next Scenario →'}</Button>
        </div>
      )}
    </Card>
  )
}
```

**Step 6: Mission page route**
```typescript
// src/app/mission/[id]/page.tsx
'use client'
import { useParams, useRouter } from 'next/navigation'
import { useProgress } from '@/hooks/useProgress'
import { MissionShell } from '@/components/mission/MissionShell'
import { getMissionById } from '@/data'

export default function MissionPage() {
  const { id } = useParams<{ id: string }>()
  const router = useRouter()
  const { completeMission } = useProgress()
  const mission = getMissionById(id)

  if (!mission) return <div className="p-8 text-gray-400">Mission not found.</div>

  const handleComplete = () => {
    completeMission(mission)
    router.push(`/reward?missionId=${mission.id}`)
  }

  return <MissionShell mission={mission} onComplete={handleComplete} />
}
```

**Step 7: Commit**
```bash
git add src/components/mission/ src/app/mission/
git commit -m "feat: add MissionShell and all four mission type components"
```

---

# SPRINT 3C — Story Panels + Reward Screen (Parallel)

> Depends on Sprint 2A + 2B + 2C. Touches `src/components/story/`, `src/components/rewards/`, `src/app/story/`, `src/app/reward/`.

## Task 13: Story panel and reward screen

**Files:**
- Create: `src/components/story/StoryPanel.tsx`
- Create: `src/components/rewards/RewardScreen.tsx`
- Create: `src/app/story/[panelId]/page.tsx`
- Create: `src/app/reward/page.tsx`

**Step 1: StoryPanel component**
```typescript
// src/components/story/StoryPanel.tsx
'use client'
import { useState } from 'react'
import { motion, AnimatePresence } from 'framer-motion'
import type { StoryPanel as StoryPanelType } from '@/types'
import { Button } from '@/components/ui'

interface StoryPanelProps {
  panel: StoryPanelType
  onComplete: () => void
}

const SPEAKER_COLORS: Record<string, string> = {
  sleepy:   'text-ow-orange',
  narrator: 'text-gray-400',
}

export function StoryPanel({ panel, onComplete }: StoryPanelProps) {
  const [index, setIndex] = useState(0)
  const slide = panel.slides[index]
  const isLast = index === panel.slides.length - 1

  return (
    <div className="min-h-screen bg-ow-dark flex items-center justify-center p-6">
      <div className="w-full max-w-xl">
        <AnimatePresence mode="wait">
          <motion.div
            key={index}
            initial={{ opacity: 0, y: 10 }}
            animate={{ opacity: 1, y: 0 }}
            exit={{ opacity: 0, y: -10 }}
            transition={{ duration: 0.3 }}
          >
            <div className="rounded-lg border border-white/10 bg-ow-dark-2 p-6 space-y-3">
              {slide.speaker && (
                <p className={`text-xs font-display uppercase tracking-widest ${SPEAKER_COLORS[slide.speaker] ?? 'text-ow-blue'}`}>
                  {slide.speaker}
                </p>
              )}
              <p className="text-lg text-white leading-relaxed">"{slide.text}"</p>
              <div className="flex justify-between items-center pt-2">
                <div className="flex gap-1">
                  {panel.slides.map((_, i) => (
                    <div key={i} className={`h-1 w-6 rounded-full transition-all ${i <= index ? 'bg-ow-blue' : 'bg-white/10'}`} />
                  ))}
                </div>
                <Button size="sm" onClick={() => isLast ? onComplete() : setIndex(i => i + 1)}>
                  {isLast ? 'Continue →' : 'Next →'}
                </Button>
              </div>
            </div>
          </motion.div>
        </AnimatePresence>
      </div>
    </div>
  )
}
```

**Step 2: Story page route**
```typescript
// src/app/story/[panelId]/page.tsx
'use client'
import { useParams, useRouter } from 'next/navigation'
import { useProgress } from '@/hooks/useProgress'
import { StoryPanel } from '@/components/story/StoryPanel'
import { chapter1Panels } from '@/data/story/chapter1'

export default function StoryPage() {
  const { panelId } = useParams<{ panelId: string }>()
  const router = useRouter()
  const { markStoryPanelViewed } = useProgress()

  const panel = chapter1Panels.find(p => p.id === panelId)
  if (!panel) return <div className="p-8 text-gray-400">Panel not found.</div>

  const handleComplete = () => {
    markStoryPanelViewed(panel.id)
    router.push('/dashboard')
  }

  return <StoryPanel panel={panel} onComplete={handleComplete} />
}
```

**Step 3: RewardScreen**
```typescript
// src/components/rewards/RewardScreen.tsx
'use client'
import { useEffect, useState } from 'react'
import { motion } from 'framer-motion'
import { useRouter } from 'next/navigation'
import { ProgressBar, Button, Badge, GlowText } from '@/components/ui'
import { useProgress } from '@/hooks/useProgress'
import { getXPToNextRank } from '@/engine/xp'
import type { Mission } from '@/types'

interface RewardScreenProps {
  mission: Mission
}

export function RewardScreen({ mission }: RewardScreenProps) {
  const router = useRouter()
  const { progress, rankUpEvent, clearRankUpEvent } = useProgress()
  const [showRankUp, setShowRankUp] = useState(false)

  useEffect(() => {
    if (rankUpEvent) {
      setShowRankUp(true)
      const t = setTimeout(() => { setShowRankUp(false); clearRankUpEvent() }, 3000)
      return () => clearTimeout(t)
    }
  }, [rankUpEvent, clearRankUpEvent])

  if (!progress) return null
  const xpInfo = getXPToNextRank(progress.xp)

  return (
    <div className="min-h-screen bg-ow-dark flex items-center justify-center p-6">
      <motion.div
        className="w-full max-w-md space-y-6 text-center"
        initial={{ opacity: 0, scale: 0.9 }}
        animate={{ opacity: 1, scale: 1 }}
        transition={{ duration: 0.4 }}
      >
        {/* Rank-up overlay */}
        {showRankUp && (
          <motion.div
            className="fixed inset-0 z-50 flex items-center justify-center bg-black/80"
            initial={{ opacity: 0 }} animate={{ opacity: 1 }} exit={{ opacity: 0 }}
          >
            <div className="text-center space-y-3">
              <motion.div
                className="font-display text-6xl font-bold text-ow-blue text-glow"
                animate={{ scale: [0.5, 1.2, 1] }}
                transition={{ duration: 0.6 }}
              >
                RANK UP
              </motion.div>
              <p className="font-display text-3xl text-ow-orange uppercase">{rankUpEvent}</p>
            </div>
          </motion.div>
        )}

        <div className="text-5xl">⭐</div>
        <h1 className="font-display text-3xl font-bold"><GlowText>Mission Complete</GlowText></h1>
        <p className="text-gray-400 italic">"{mission.title}"</p>

        <motion.div
          className="font-display text-4xl text-ow-orange"
          initial={{ scale: 0 }} animate={{ scale: 1 }}
          transition={{ type: 'spring', delay: 0.2 }}
        >
          +{mission.xpReward} XP
        </motion.div>

        {xpInfo && (
          <div className="space-y-1">
            <ProgressBar
              value={(xpInfo.current / xpInfo.required) * 100}
              label={`Progress to ${xpInfo.rank.toUpperCase()}`}
              height="lg"
              animated
            />
          </div>
        )}

        <div className="flex gap-3 justify-center">
          <Button variant="secondary" onClick={() => router.push('/map')}>Skill Map</Button>
          <Button glow onClick={() => router.push('/dashboard')}>Continue →</Button>
        </div>
      </motion.div>
    </div>
  )
}
```

**Step 4: Reward page route**
```typescript
// src/app/reward/page.tsx
'use client'
import { useSearchParams } from 'next/navigation'
import { Suspense } from 'react'
import { RewardScreen } from '@/components/rewards/RewardScreen'
import { getMissionById } from '@/data'

function RewardContent() {
  const params = useSearchParams()
  const missionId = params.get('missionId') ?? ''
  const mission = getMissionById(missionId)
  if (!mission) return <div className="p-8 text-gray-400">Mission not found.</div>
  return <RewardScreen mission={mission} />
}

export default function RewardPage() {
  return <Suspense><RewardContent /></Suspense>
}
```

**Step 5: Commit**
```bash
git add src/components/story/ src/components/rewards/ src/app/story/ src/app/reward/
git commit -m "feat: add StoryPanel viewer and RewardScreen with rank-up animation"
```

---

# SPRINT 4 — Integration (Serial, do last)

## Task 14: Landing / onboarding page

**Files:**
- Modify: `src/app/page.tsx`

**Step 1: Write landing page**
```typescript
// src/app/page.tsx
'use client'
import { useEffect } from 'react'
import { useRouter } from 'next/navigation'
import { motion } from 'framer-motion'
import { Button, GlowText } from '@/components/ui'
import { loadProgress } from '@/engine/storage'

export default function Home() {
  const router = useRouter()

  useEffect(() => {
    const p = loadProgress()
    if (p.completedMissions.length > 0) router.replace('/dashboard')
  }, [router])

  const startJourney = () => {
    router.push('/story/panel-ch01-intro')
  }

  return (
    <div className="min-h-screen bg-ow-dark flex items-center justify-center p-6">
      <div className="text-center max-w-lg space-y-6">
        <motion.div
          initial={{ opacity: 0, y: -20 }}
          animate={{ opacity: 1, y: 0 }}
          transition={{ duration: 0.6 }}
        >
          <h1 className="font-display text-6xl font-bold tracking-wider">
            <GlowText>WATCHPOINT</GlowText>
          </h1>
          <p className="mt-3 text-gray-400 text-lg">Learn Overwatch. Follow Sleepy's Journey.</p>
        </motion.div>

        <motion.div
          className="space-y-2 text-sm text-gray-500"
          initial={{ opacity: 0 }} animate={{ opacity: 1 }} transition={{ delay: 0.4 }}
        >
          <p>📖 Story-driven missions that teach real skills</p>
          <p>🎯 10 Boot Camp missions to master the fundamentals</p>
          <p>⭐ XP, ranks, and cosmetic rewards</p>
        </motion.div>

        <motion.div initial={{ opacity: 0 }} animate={{ opacity: 1 }} transition={{ delay: 0.7 }}>
          <Button glow size="lg" onClick={startJourney}>
            Begin Sleepy's Story →
          </Button>
        </motion.div>
      </div>
    </div>
  )
}
```

**Step 2: Commit**
```bash
git add src/app/page.tsx
git commit -m "feat: add landing page with auto-redirect for returning players"
```

---

## Task 15: Profile page

**Files:**
- Create: `src/app/profile/page.tsx`

**Step 1: Write profile page**
```typescript
// src/app/profile/page.tsx
'use client'
import { useProgress } from '@/hooks/useProgress'
import { Card, Badge, GlowText, ProgressBar } from '@/components/ui'
import { getXPToNextRank } from '@/engine/xp'
import { allMissions, campaigns } from '@/data'

export default function ProfilePage() {
  const { progress } = useProgress()
  if (!progress) return null

  const xpInfo = getXPToNextRank(progress.xp)
  const totalMissions = allMissions.length
  const completedCount = progress.completedMissions.length

  return (
    <div className="min-h-screen bg-ow-dark p-6">
      <div className="mx-auto max-w-2xl space-y-6">
        <div className="text-center space-y-2">
          <h1 className="font-display text-4xl font-bold"><GlowText>{progress.username}</GlowText></h1>
          <Badge label={progress.rank.toUpperCase()} color="blue" />
          {progress.activeTitle && <p className="text-gray-400 text-sm">"{progress.activeTitle}"</p>}
        </div>

        <Card>
          <p className="font-display text-sm text-gray-400 mb-3">PROGRESSION</p>
          <div className="space-y-3">
            <div className="flex justify-between text-sm">
              <span className="text-gray-400">Total XP</span>
              <span className="text-ow-blue font-display">{progress.xp.toLocaleString()} XP</span>
            </div>
            <div className="flex justify-between text-sm">
              <span className="text-gray-400">Missions Complete</span>
              <span className="text-white font-display">{completedCount} / {totalMissions}</span>
            </div>
            <div className="flex justify-between text-sm">
              <span className="text-gray-400">Campaigns Complete</span>
              <span className="text-white font-display">{progress.completedCampaigns.length} / {campaigns.length}</span>
            </div>
            {xpInfo && <ProgressBar value={(xpInfo.current / xpInfo.required) * 100} label={`→ ${xpInfo.rank.toUpperCase()}`} height="md" />}
          </div>
        </Card>

        {progress.earnedCosmetics.length > 0 && (
          <Card>
            <p className="font-display text-sm text-gray-400 mb-3">EARNED BADGES</p>
            <div className="flex flex-wrap gap-2">
              {progress.earnedCosmetics.map(id => (
                <Badge key={id} label={id.replace('badge-', '').replace(/-/g, ' ').toUpperCase()} color="orange" size="sm" />
              ))}
            </div>
          </Card>
        )}
      </div>
    </div>
  )
}
```

**Step 2: Commit**
```bash
git add src/app/profile/
git commit -m "feat: add profile page showing stats, rank, and earned badges"
```

---

## Task 16: Navigation + campaign list page

**Files:**
- Create: `src/components/Nav.tsx`
- Create: `src/app/campaign/[id]/page.tsx`
- Modify: `src/app/layout.tsx`

**Step 1: Navigation bar**
```typescript
// src/components/Nav.tsx
'use client'
import Link from 'next/link'
import { usePathname } from 'next/navigation'

const links = [
  { href: '/dashboard', label: 'Home' },
  { href: '/map',       label: 'Skill Map' },
  { href: '/profile',   label: 'Profile' },
]

export function Nav() {
  const pathname = usePathname()
  if (pathname === '/') return null

  return (
    <nav className="fixed bottom-0 left-0 right-0 z-40 border-t border-white/10 bg-ow-dark-2/95 backdrop-blur">
      <div className="mx-auto flex max-w-md justify-around py-2">
        {links.map(link => (
          <Link key={link.href} href={link.href}
            className={`flex flex-col items-center px-4 py-1 font-display text-xs transition-colors
              ${pathname.startsWith(link.href) ? 'text-ow-blue' : 'text-gray-500 hover:text-gray-300'}`}>
            {link.label}
          </Link>
        ))}
      </div>
    </nav>
  )
}
```

**Step 2: Campaign page**
```typescript
// src/app/campaign/[id]/page.tsx
'use client'
import { useParams, useRouter } from 'next/navigation'
import { useProgress } from '@/hooks/useProgress'
import { campaigns } from '@/data/campaigns'
import { allMissions } from '@/data'
import { Card, Button, Badge, GlowText, ProgressBar } from '@/components/ui'

export default function CampaignPage() {
  const { id } = useParams<{ id: string }>()
  const router = useRouter()
  const { progress } = useProgress()
  const campaign = campaigns.find(c => c.id === id)

  if (!campaign || !progress) return null
  const missions = allMissions.filter(m => m.campaignId === id)
  const completedCount = missions.filter(m => progress.completedMissions.includes(m.id)).length

  return (
    <div className="min-h-screen bg-ow-dark p-6 pb-20">
      <div className="mx-auto max-w-xl space-y-4">
        <div>
          <h1 className="font-display text-3xl font-bold"><GlowText>{campaign.title}</GlowText></h1>
          <p className="text-gray-400 mt-1">{campaign.description}</p>
          <ProgressBar value={(completedCount / missions.length) * 100} label={`${completedCount}/${missions.length} complete`} height="md" />
        </div>

        {missions.map((mission, i) => {
          const isDone = progress.completedMissions.includes(mission.id)
          const isUnlocked = i === 0 || progress.completedMissions.includes(missions[i - 1].id)
          return (
            <Card key={mission.id} locked={!isUnlocked}>
              <div className="flex items-start justify-between gap-3">
                <div className="flex-1">
                  <div className="flex items-center gap-2 mb-1">
                    <span className="font-display text-xs text-gray-500">{String(i + 1).padStart(2, '0')}</span>
                    <h3 className="font-display font-semibold">{isDone ? '✓ ' : ''}{mission.title}</h3>
                  </div>
                  <div className="flex gap-1 flex-wrap">
                    {mission.types.map(t => <Badge key={t} label={t} size="sm" />)}
                    <span className="text-xs text-gray-500 self-center">~{mission.estimatedMinutes}m · {mission.xpReward}xp</span>
                  </div>
                </div>
                {isUnlocked && !isDone && (
                  <Button size="sm" onClick={() => router.push(`/mission/${mission.id}`)}>Start</Button>
                )}
                {!isUnlocked && <span className="text-gray-600 text-sm">🔒</span>}
              </div>
            </Card>
          )
        })}
      </div>
    </div>
  )
}
```

**Step 3: Add Nav to layout**
```typescript
// src/app/layout.tsx — add Nav import and usage
import { Nav } from '@/components/Nav'
// Inside <body>: add <Nav /> before {children}
```

**Step 4: Commit**
```bash
git add src/components/Nav.tsx src/app/campaign/ src/app/layout.tsx
git commit -m "feat: add navigation bar and campaign mission list page"
```

---

## Task 17: Final verification and deploy config

**Step 1: Verify build passes**
```bash
npm run build
```
Expected: No TypeScript errors, no build failures.

**Step 2: Fix any type errors** — address errors one at a time.

**Step 3: Create `.claude/launch.json`**
```json
{
  "version": "0.0.1",
  "configurations": [
    {
      "name": "Watchpoint Dev",
      "runtimeExecutable": "npm",
      "runtimeArgs": ["run", "dev"],
      "port": 3000
    }
  ]
}
```

**Step 4: Commit and push**
```bash
git add -A
git commit -m "feat: complete Watchpoint MVP — Boot Camp campaign, mission system, story panels, rewards"
git push origin master
```

---

## Parallel Execution Summary

| Sprint | Tasks | Can run in parallel with |
|--------|-------|--------------------------|
| Sprint 1 | Tasks 1–3 | Serial — do first |
| Sprint 2A | Tasks 4–7 | Sprint 2B, Sprint 2C |
| Sprint 2B | Tasks 8–9 | Sprint 2A, Sprint 2C |
| Sprint 2C | Task 10 | Sprint 2A, Sprint 2B |
| Sprint 3A | Task 11 | Sprint 3B, Sprint 3C |
| Sprint 3B | Task 12 | Sprint 3A, Sprint 3C |
| Sprint 3C | Task 13 | Sprint 3A, Sprint 3B |
| Sprint 4 | Tasks 14–17 | Serial — do last |
