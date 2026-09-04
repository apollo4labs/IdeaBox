# 🔁 HabitStack

> A micro-habits app focused on building routines in 2-minute increments with gamified streaks and social accountability.

**Status:** Concept

---

## One-Liner
_A micro-habits app that turns any routine into a stack of 2-minute daily actions, combining low-friction habit loops with gamified streaks and optional social accountability._

---

## 📋 Overview
Most habit apps fail on an accessibility paradox: they ask people who can't maintain a habit to commit to big, intimidating actions. HabitStack inverts this by decomposing every goal into **2-minute micro-actions** that are nearly impossible to skip, then stacking them into visible progress. The "stack" is both the core mechanic (actions stack up over the day) and the metaphor (small bricks build a routine).

Where competitors push heavy journaling, long streaks-with-forgiveness, and private goal-setting, HabitStack optimizes for **completion rate** — the metric that actually predicts long-term behavior change — and pairs it with optional social accountability for people who thrive on not letting a team down.

---

## ❓ Problem
Habit formation tools overwhelmingly fail for three reasons:

### Pain Points
- **Too much friction per action** — apps ask for 30–60 minute sessions or detailed logging, which breaks the cadence on day one or on any busy day.
- **Streak anxiety > motivation** — "Don't break the chain" creates guilt; one miss kills momentum and users abandon the whole app.
- **No accountability for those who want it** — solo apps have no way to make a commitment visible to others, so social users lose a powerful motivator.

### Current Alternatives
- **Streak-only apps (Duolingo-style, Loop Habit)** — motivation dies when the streak breaks; no recovery path.
- **Full journaling apps (Day One, journaling templates)** — too heavy for a daily 2-minute commitment.
- **To-do managers (Todoist, Things)** — can track tasks but aren't built around habit psychology or streaks.
- **Accountability groups (message-based)** — unstructured, noisy, no shared progress visibility.

### The Gap
No app combines **guaranteed-small actions (2 min)** with **gamified stacks that reward partial days** and **structured social accountability** — the three levers that together make habits stick without relying on constant motivation.

---

## 💡 Solution
HabitStack makes the habit loop trivially easy to start and hard to quietly abandon.

### Core Feature 1 — 2-Minute Micro-Actions
Every user goal (e.g. "exercise," "write," "read") is decomposed into a suggested stack of 2-minute actions. Completion requires *starting* the action, not finishing a long session — an open-ended timer lets a user extend past 2 minutes if they're in flow. The philosophy: **start is the habit; duration follows.**

### Core Feature 2 — The Stack Streak
Instead of a fragile daily streak, HabitStack tracks a **stack score**: each completed 2-minute action adds a brick. A day is "good" if you complete at least one action per stacked goal; partial days still add bricks and never reset progress retroactively. Streaks become a *consecutive-good-days* metric with a one-time "recovery brick" allowance, killing streak anxiety.

### Core Feature 3 — Social Squads (opt-in)
Users can join small accountability squads (2–8 people). Squad members see each other's anonymized completion state *(on-track / slipped)*, and a lightweight daily "nudge" — no persistent stream of comparison, just a gentle accountability check-in. Squads are fully optional; solo mode is the default.

### Core Feature 4 — Smart Suggestion Engine
Based on completion patterns, the app suggests when to schedule each action (morning vs. evening) and gradually *stacks* adjacent actions into a routine — e.g. "water → stretch → journal" becomes one 6-minute morning stack.

### Core Feature 5 — Offline-First & Cross-Device
Actions and streaks work entirely offline; sync happens silently when connectivity returns. Mobile-first with a lightweight web view, so a user never loses a completion to a dead zone.

---

## 👥 Target Users

| Segment | Description | Size |
|---------|-------------|------|
| Self-improvers who've failed at habits before | People burned by past streak-based apps looking for a gentler on-ramp | Very large |
| Busy professionals | Time-poor users who need proof that 2 minutes still counts | Very large |
| Students building routines | Campus users with fragmented schedules | Large |
| Accountability-driven people | Users who want a small, trusted circle to keep them honest | Medium |

### User Personas
**Persona 1 — "The Burned Streaker"**
_Sam, 29, tried Duolingo-style streaks twice and quit after the first miss. Wants progress that doesn't punish a bad day. → Loves recovery bricks and stack scores._

**Persona 2 — "The Time-Poor Professional"**
_Priya, 34, consultant with chaotic days. Can never find 30 minutes. → 2-minute actions fit between meetings; the suggestion engine re-times actions for her worst days._

**Persona 3 — "The Accountable Teammate"**
_Jonas, 24, graduate student who works better when others expect it of him. → Joins a squad, sees others' on-track states, relies on gentle nudges._

---

## 🏗️ Architecture

A mobile-first offline-first app with a sync backend and a predictable API.

### High-Level Architecture
```
[Android / iOS app]  ──local storage──▶  [On-device habit engine]
        │                                        │
        ▼ sync (silent)                          ▼
[Mobile Sync API] ◀──▶ [Postgres]  ◀──▶ [Squad/Accountability service]
        │                                        │
        ▼ (optional)                              ▼ (optional, later)
[Push notification worker]                [Opponent-free social graph]
```

### Key Design Decisions
- **Local-first:** habit engine, stack scoring, and streak logic live on-device so they work offline. Backend is a sync target, not a source of truth for daily interaction.
- **Silent sync:** changes propagate as background events; conflict resolution is last-write-wins per action with timestamps.
- **Squads are small & anonymized:** squad membership is the only cross-user data; progress is shared as completion-state booleans, not detailed logs.

### Tech Stack
- **Frontend (mobile):** Flutter (single codebase for Android + iOS)
- **Local storage:** SQLite (via drift) on-device
- **Backend API:** Node.js (Fastify) or Go lightweight service
- **Database:** PostgreSQL
- **Realtime (squads):** WebSockets or Firebase Realtime (evaluate at MVP; a simple REST poll is enough to start)
- **Push/notifications:** native push (FCM / iOS Push), background workers
- **Infrastructure:** managed VM or Fly.io; $0–20/month to start

---

## 💰 Business Model

### Revenue Streams
- **Freemium:** core habit engine, stacks, and streaks free forever.
- **HabitStack Pro (subscription):** unlimited squads, smart suggestion engine, advanced analytics, themes, cross-device history export.

### Pricing Tiers
| Tier | Price | Features |
|------|-------|----------|
| Free | $0 | Micro-actions, stacks, streaks, 1 squad, offline sync |
| Pro | $4–6/mo | Unlimited squads, suggestion engine, analytics, export, themes |
| Team/School | Custom | Bulk licensing for teams or school programs |

### Unit Economics
- CAC: ~$0.50–2 (organic + app-store discovery)
- LTV: Pro ~$50–72/yr; free → Pro conversion ~4–7%
- Target margin: >85% (infra cost is a tiny fixed cost)

---

## 🚀 MVP Scope

### Must-Have (MVP)
- [ ] Add goals and decompose into 2-minute micro-actions
- [ ] One-tap start with 2-minute timer (extendable)
- [ ] Local stack score + consecutive-good-days streak
- [ ] Recovery brick allowance (no streak-reset anxiety)
- [ ] Full offline use with silent sync
- [ ] One squad, anonymized completion states, daily nudge
- [ ] Reminder notifications at user-set times

### Should-Have (Post-MVP)
- [ ] Smart suggestion engine for action timing and stacking
- [ ] Cross-device web view
- [ ] Analytics dashboard (personal trends)
- [ ] Multiple squads + roles/moderators
- [ ] Export / backup

### Nice-to-Have
- [ ] Granular sharing (share a goal's progress publicly)
- [ ] Habit widget / watch companion app
- [ ] Challenges between squads
- [ ] Localization & accessibility pass

---

## 📈 Success Metrics

### North Star Metric
**Completed micro-actions per active user per week** — the direct proxy for whether the habit loop is working.

### KPIs
- D1 retention: target >60%
- D30 retention: target >40%
- Median time-to-first-completion after install: <5 minutes
- Streak recovery rate (users who miss a day but return within 48h): >50%
- Squad users' weekly completion rate vs. solo users (aim: +15%)

---

## 🔄 Development Roadmap

### Phase 1: Core Loop (Weeks 1–4)
- [ ] Flutter scaffold, local habit engine, SQLite storage
- [ ] 2-minute timer + stack scoring
- [ ] Reminders + basic notifications
- [ ] Offline-first sync foundation

### Phase 2: Streaks & Recovery (Weeks 5–8)
- [ ] Consecutive-good-days streak
- [ ] Recovery brick logic
- [ ] Backend API + Postgres for sync
- [ ] Onboarding flow ("your first micro-habit in 2 min")

### Phase 3: Squads & Accountability (Weeks 9–12)
- [ ] Squad creation/joining
- [ ] Anonymized completion states + daily nudge
- [ ] Web view
- [ ] Beta with 100 users, iterate on retention

### Phase 4: Monetization & Polish (Weeks 13–16)
- [ ] Pro tier, billing integration
- [ ] Smart suggestion engine
- [ ] Analytics dashboards
- [ ] Accessibility + localization pass

---

## ⚠️ Risks & Challenges

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| App fatigue — users churn after novelty | High | High | Focus entire product on completion rate; suggestion engine re-times actions; squads add social hooks |
| "2 minutes is too trivial" skepticism | Medium | Medium | Position as *start is the habit*; allow extending sessions; show weekly aggregate wins |
| Squad feature adds pressure / toxicity | Medium | Medium | Keep squads opt-in and small; anonymized states; clear opt-out; moderation tools |
| Offline sync conflicts | Low | Medium | Timestamped last-write-wins; transparent conflict log |
| Competition from habit giants | High | Medium | Differentiate on micro-actions + stack score + squads; niche positioning |

---

## 📁 Related Files

- `diagrams/architecture.plantuml` — System architecture
- `diagrams/user-flow.plantuml` — Onboarding, daily loop, squad flow

---

*Last updated: 2026-09-04*