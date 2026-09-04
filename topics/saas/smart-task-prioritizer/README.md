# 🧠 Smart Task Prioritizer

> An intelligent task management tool that uses machine learning to automatically prioritize your daily tasks based on deadlines, importance, energy levels, and dependency graphs.

**Status:** Concept

---

## One-Liner
_An AI-powered to-do app that turns a chaotic task list into an optimally ordered daily plan — weighing deadlines, importance, your energy levels, and task dependencies automatically._

---

## 📋 Overview
People don't lack task lists; they lack good ordering. Plain to-do apps store tasks and let users manually guess what to do next — a cognitive load that most people handle poorly, especially under stress. Smart Task Prioritizer is a task manager where the **prioritization is the product**: a machine-learning model continuously re-ranks today's tasks into the single best sequence, factoring in hard deadlines, stated importance, personal energy rhythms, and dependency graphs (what must finish before what).

The result is a "just do the next thing" experience: no more 9 AM paralysis over an unordered list. It's personal, adaptive, and gets better as it learns how you actually work.

---

## ❓ Problem
Knowledge workers and students drown in unprioritized task lists:

### Pain Points
- **The ordering tax** — Deciding *what to do next* is itself exhausting; studies estimate workers spend significant time each day on task-switching and re-planning.
- **Deadline blindness** — Without explicit priority logic, urgent-but-not-important and important-but-not-urgent tasks get conflated.
- **Energy mismatch** — Everyone has low-energy and high-energy periods, but most apps treat all hours as equal.
- **Dependency errors** — Starting a task that depends on unfinished work wastes effort and causes rework.

### Current Alternatives
- **Manual to-do apps (Todoist, Things, TickTick)** — store tasks but leave ordering to the user.
- **Priority matrix apps (Eisenhower)** — static, manual, don't adapt over time.
- **Calendars/time-blocking** — rigid; don't model dependencies or energy.
- **Generic AI assistants** — answer questions but don't maintain an adaptive priority model of your work.

### The Gap
No mainstream tool **automatically and continuously** computes task order from a learned model of deadlines, importance, energy, and dependencies — while staying simple enough for a daily-driver.

---

## 💡 Solution
A task manager with a prioritization engine at its core, wrapped in an opinionated, low-friction UI.

### Core Feature 1 — ML Priority Ranking
A ranker scores every task from features: deadline proximity, stated importance, estimated effort, dependency block status, and your learned energy patterns. It re-ranks continuously as things change (new tasks, shifting deadlines, task completion). Output: a single "do this next" queue.

### Core Feature 2 — Energy-Aware Scheduling
The model learns your personal energy curve (from self-reported check-ins or activity patterns) and slots deep-work tasks into high-energy windows and shallow/administrative tasks into low-energy ones. Users can override freely; the model learns from overrides.

### Core Feature 3 — Dependency Graphs
Tasks can be linked ("X blocks Y"). The ranker never suggests a blocked task before its blockers, and surfaces critical-path warnings when a blocker is late — preventing wasted effort and rework.

### Core Feature 4 — Stress-Aware Buffering
On heavy days, the app proposes cutting scope: it identifies the tasks to *defer* or *delegate* so the plan stays realistic, rather than presenting an impossible list that breeds guilt.

### Core Feature 5 — Explainable Recommendations
Every suggestion comes with a one-line reason ("High urgency, fires tomorrow" / "Low energy window, low-focus task"). Transparency builds trust and keeps the user in control — auto-prioritize can be toggled off at any time.

---

## 👥 Target Users

| Segment | Description | Size |
|---------|-------------|------|
| Knowledge workers | People with many simultaneous projects and deadlines | Very large |
| Students | Multiple courses, assignments, exams, group work | Large |
| Freelancers / consultants | Clients, billable hours, competing priorities | Large |
| Managers & leaders | Delegation, review, heavy dependency load | Large |

### User Personas
**Persona 1 — "The Overloaded Manager"**
_Daniel, 38, manages a team and 40+ open tasks across projects. Struggles to order his day. → Opens the app and just starts the top item; the model respects blockers and his morning deep-work window._

**Persona 2 — "The Deadline-Driven Student"**
_Aisha, 22, juggling five courses. Keeps missing non-urgent-but-important work. → The ranker surfaces creeping deadlines before they become emergencies._

**Persona 3 — "The Freelancer"**
_Nick, 29, juggles retainers and one-off gigs. → Dependency + effort awareness stops him from double-booking; energy-aware slots protect his creative hours._

---

## 🏗️ Architecture

A task store with a ranking service, learning layer, and sync clients.

### High-Level Architecture
```
[Web / Mobile / Desktop apps]
        │
        ▼
[Task/Project API] ──▶ [Postgres]
        │
        ▼
[Prioritization Service] ──▶ [Ranking model (ML)]
        │                            │
        ▼                            ▼
[Dependency engine]          [Learning pipeline (from overrides/outcomes)]
```

### Key Design Decisions
- **Ranking is a service, not a batch job** — re-ranked on every relevant event (task add/complete/deadline change) for responsiveness.
- **On-device + cloud hybrid for the model** — fast scoring on-device; heavier learning in the cloud.
- **Explainable outputs** — ranker returns feature attributions for each recommendation so the UI can show "why."

### Tech Stack
- **Frontend:** React/Next.js (web), Flutter (mobile), native-light desktop via web wrapper
- **Backend API:** Node.js (Fastify) or Go
- **Database:** PostgreSQL (tasks, deps, overrides, events)
- **ML ranker:** gradient-boosted model (LightGBM/XGBoost) scoring on event; small on-device scorer (e.g., ONNX runtime) for latency
- **Learning pipeline:** scheduled retraining from interaction logs
- **Realtime:** WebSockets for live re-ordering pushes
- **Infrastructure:** managed VM or Fly.io; $20–60/month at launch

---

## 💰 Business Model

### Revenue Streams
- **Freemium:** task management, dependencies, and manual priority always free.
- **Smart Prioritizer Pro (subscription):** automated ML ranking, energy-aware scheduling, learning/personalization, advanced analytics.

### Pricing Tiers
| Tier | Price | Features |
|------|-------|----------|
| Free | $0 | Unlimited tasks, projects, dependencies, manual priority |
| Pro | $6–9/mo | Automated priority ranking, energy scheduling, personalization, analytics |
| Team | $8–12/user/mo | Shared projects, team dependencies, assignee awareness |

### Unit Economics
- CAC: ~$1–3 (organic + content marketing)
- LTV: Pro ~$80–110/yr; conversion ~5–8%
- Target margin: >80%

---

## 🚀 MVP Scope

### Must-Have (MVP)
- [ ] Add/organize tasks, projects, due dates, importance, effort
- [ ] Dependency links ("X blocks Y")
- [ ] Manual priority sort (baseline UX)
- [ ] Simple heuristic ranker v1 (deadline + importance + deps)
- [ ] "Do this next" queue UI
- [ ] Web app + mobile app
- [ ] Sync across devices

### Should-Have (Post-MVP)
- [ ] ML ranker with energy modeling
- [ ] Learning from user overrides/outcomes
- [ ] Explainable "why this order" lines
- [ ] Stress-aware defer/delegate suggestions
- [ ] Realtime re-ordering pushes

### Nice-to-Have
- [ ] Calendar/Google Calendar two-way sync
- [ ] Time-blocking suggestions
- [ ] Team/assignee-aware ranking
- [ ] Reporting & weekly review digest

---

## 📈 Success Metrics

### North Star Metric
**Tasks completed per week per active user** — the ultimate proof the ordering is helping, not just the app being open.

### KPIs
- Time-to-first-ranked-task after signup: <2 minutes
- No-miss rate on due dates (on-time completions): target +30% vs. manual baseline in beta
- Dependency-error rate (started blocked task): target near 0
- D30 retention: target >40%
- Override rate (user changes suggested order): target <20% (trust) but decreasing

---

## 🔄 Development Roadmap

### Phase 1: Task Foundation (Weeks 1–4)
- [ ] Task/project model, due dates, importance, effort
- [ ] Dependency links
- [ ] Manual priority sort
- [ ] Web app + sync backend

### Phase 2: Heuristic Ranker (Weeks 5–8)
- [ ] v1 ranker: deadline + importance + dependency awareness
- [ ] "Do this next" queue UI
- [ ] Mobile app
- [ ] Onboarding + 50-user pilot

### Phase 3: ML & Personalization (Weeks 9–12)
- [ ] Logging pipeline (events, overrides, outcomes)
- [ ] ML ranker with energy modeling
- [ ] Learning from overrides
- [ ] Explainable recommendation lines
- [ ] A/B test vs. heuristic ranker

### Phase 4: Scale & Monetize (Weeks 13–16)
- [ ] Realtime re-order pushes
- [ ] Pro tier + billing
- [ ] Stress-aware buffering
- [ ] Calendar integration, reporting, public launch

---

## ⚠️ Risks & Challenges

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| "I don't trust the AI" skepticism | High | High | Explainable reasons, full manual override, show confidence, gradual onboarding |
| Model personalization needs data before it's useful | High | Medium | Ship strong heuristic default; learn quickly from overrides; don't gate core UX on ML |
| Cold-start for new users | Medium | Medium | Sensible defaults (Eisenhower-style weights); optional quick survey |
| Feature creep into a generic to-do app | Medium | Medium | Keep ranking as the hero; defer/complex features |
| Competition from task giants adding AI | High | Medium | Move fast on dependency+energy+explainability differentiators; niche pro positioning |

---

## 📁 Related Files

- `diagrams/architecture.plantuml` — System architecture and ranking pipeline
- `diagrams/user-flow.plantuml` — Task entry, ranking, override, weekly review

---

*Last updated: 2026-09-04*