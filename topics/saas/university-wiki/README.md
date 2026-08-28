# 🏫 University Wiki

**Status:** Concept

## One-Liner
_A student-run, mediawiki-based knowledge hub for an entire university — notes, lectures, exams, and a scraper bot that keeps everything up to date from the official university page._

---

## 📋 Overview

A single wiki per university where students collaboratively document everything from their first lecture to their doctorate. Built on MediaWiki with markdown support, hosted on a cheap VPS, and exposed via Cloudflare Tunnel. A companion bot continuously scrapes the university's official page for the latest updates — timetables, exam schedules, policy changes, course registrations — and surfaces them automatically.

The project is run by students, for students. Free. Ad-free. No paywall. Just a shared brain for the university.

---

## ❓ Problem

Universities are notoriously bad at centralizing information. Students have to juggle:

### Pain Points
- 📰 **Scattered info** — timetables, exam dates, course descriptions, and announcements are spread across a dozen webpages, PDFs, and Facebook groups
- 📝 **Lost knowledge** — lecture notes, past exams, and study guides from previous years vanish when students graduate or delete old posts
- 🔄 **Stale content** — official university pages are rarely updated by the time students need them; a 2022 exam schedule is useless in 2025
- 🤝 **No collaboration layer** — there's no structured way to share notes, discuss topics, or build a collective resource
- 💸 **Costly alternatives** — commercial platforms charge for hosting, premium features, or visibility

### Current Alternatives
| Alternative | Gap |
|-------------|-----|
| University LMS (Moodle, etc.) | Clunky, institution-controlled, no community editing |
| Facebook / Telegram groups | Unstructured, no search, content disappears |
| Personal blogs / Notion | Individual effort, no critical mass |
| GitHub repos | Too technical for most students |

---

## 💡 Solution

A student-owned MediaWiki instance that becomes the single source of truth for everything a student needs during their entire academic journey.

### Core Feature 1 — Full Wiki with Markdown
MediaWiki backend with markdown rendering. Students can create pages for courses, lectures, exams, books, and general knowledge. Searchable, version-controlled, and linkable.

### Core Feature 2 — Media Library
Upload and organize lecture slides, recorded lectures, PDFs, textbooks, and past exams per course/year/semester. Tagged and searchable.

### Core Feature 3 — Forums & Discussion
Course-specific forums and general-knowledge threads. Students can ask questions, share tips, and help each other.

### Core Feature 4 — Scraper Bot
A bot that continuously scrapes the university's official page (timetables, exam schedules, registration deadlines, policy updates) and auto-updates relevant wiki pages. Alerts students when something changes.

### Core Feature 5 — Lifecycle Coverage
Content organized by academic year, semester, and course — from orientation-week basics to doctoral-research resources. Everything a student needs at every stage.

---

## 👥 Target Users

| Segment | Description | Size |
|---------|-------------|------|
| Primary | Current students (undergrad + master) of one university | 500–50,000 |
| Secondary | Alumni (for historical notes/exams), faculty (for resource sharing) | Variable |
| Tertiary | Prospective students (admission info, freshman guides) | Variable |

### User Personas
**Persona 1: The Overwhelmed Freshman**
_Name:_ Alex, first-year CS student<br>
_Goals:_ Find syllabi, buy the right books, understand the grading system<br>
_Frustrations:_ Nobody told them anything in orientation; info is scattered across 10+ sites

**Persona 2: The Exam-Cramming Senior**
_Name:_ Maria, final-year mechanical engineering student<br>
_Goals:_ Find past exams with answers, study notes from A-year students, exam schedule changes<br>
_Frustrations:_ Old exam papers aren't shared; the official schedule was updated 3 days ago

**Persona 3: The Community Builder**
_Name:* Jonas, 3rd year, runs the student Discord<br>
_Goals:_ Centralize information, reduce repetitive questions, build a lasting resource<br>
_Frustrations:* No good wiki exists; Facebook groups get noisy and cluttered

---

## 🏗️ Architecture

Student-run, lean infrastructure. Cheap to run. Easy to deploy.

### High-Level Architecture
```
┌─────────────────┐      ┌──────────────────┐      ┌──────────────────┐
│   Cloudflare    │─────▶│   VPS (Hetzner/   │─────▶│   PostgreSQL      │
│   Tunnel        │      │   Equinix)        │      │   + MediaWiki     │
└─────────────────┘      │   Nginx           │      └──────────────────┘
                         └────────┬─────────┘               │
                                  │                         ▼
                         ┌────────▼─────────┐      ┌──────────────────┐
                         │  Scraper Bot      │─────▶│  University       │
                         │  (Python/Playwright)│    │  Official Pages   │
                         └──────────────────┘      └──────────────────┘
```

### Tech Stack
- **Wiki Engine:** MediaWiki with [VisualEditor](https://www.mediawiki.org/wiki/Extension:VisualEditor) + [Markdown extension](https://www.mediawiki.org/wiki/Extension:Pagedown)
- **Backend:** PHP (MediaWiki native), Python for the scraper bot
- **Database:** PostgreSQL (MediaWiki-compatible)
- **File Storage:** Local + S3-compatible (MinIO) for media uploads, or just the VPS disk for a single-server setup
- **Infrastructure:** Cheap VPS (Hetzner/Equinix ~€4/mo), Cloudflare Tunnel for HTTPS (free), Cloudflare CDN
- **Scraper Bot:** Python + Playwright (for JS-rendered pages) or BeautifulSoup (static pages), cron job or systemd timer
- **Search:** MediaWiki built-in + [CirrusSearch](https://www.mediawiki.org/wiki/Extension:CirrusSearch) with Elasticsearch (optional for larger wikis)
- **Auth:** MediaWiki native accounts + optional LDAP/SSO if the university provides one
- **Monitoring:** UptimeRobot (free), Healthchecks.io for the bot

---

## 💰 Business Model

This is a community project. No ads. No paywall. But it needs to survive.

### Revenue Streams
- **University sponsorship** — pitch to the student council or CS department for funding (~€100–500/year for VPS + domain)
- **Donations** — optional one-time donations via GitHub Sponsors or Ko-fi
- **Merch** — university-themed stickers/t-shirts for supporters (community-driven)
- **Premium tier** (long-term) — ad-free, extra storage, API access, custom themes for €2–3/month

### Pricing Tiers
| Tier | Price | Features |
|------|-------|----------|
| Free | $0 | Full wiki, forums, media upload, basic bot |
| Supporter | €3/mo | Ad-free, priority support, extra storage, API access |
| University | Custom | Dedicated instance, SSO, admin dashboard for faculty |

### Unit Economics
- CAC: ~€0 (organic, student-to-student word of mouth)
- LTV: Free tier has infinite reach; Supporter tier ~€36/year
- Target margin: >90% (VPS cost is the main expense)

---

## 🚀 MVP Scope

### Must-Have (MVP)
- [ ] MediaWiki instance on a VPS with Cloudflare Tunnel (HTTPS)
- [ ] Markdown rendering for easy editing
- [ ] Core wiki pages: course list, semester structure, exam schedule, books
- [ ] File uploads for lecture notes, past exams
- [ ] Basic MediaWiki user accounts and editing
- [ ] Scraper bot that watches the university page and creates/updates wiki pages for key info (timetable, exams, deadlines)
- [ ] Forum/category system for Q&A

### Should-Have (Post-MVP)
- [ ] Course-specific sub-wikis with auto-generated templates
- [ ] Search improvements (CirrusSearch)
- [ ] Notification system for bot-detected changes
- [ ] Mobile-friendly theme (MinervaNeon or Vector 2022)
- [ ] Alumni access (read-only for historical content)

### Nice-to-Have
- [ ] Video embedding/hosting for recorded lectures
- [ ] Book recommendation engine / crowdsourced textbook list
- [ ] Integration with university SSO/LDAP
- [ ] API for third-party apps (study planner, flashcard bots)
- [ ] Multi-university federation (share content across partner universities)

---

## 📈 Success Metrics

### North Star Metric
Active pages created per semester — the more content, the more students find and use it.

### KPIs
- Pages created in first semester: >500
- Unique editors (students who create/edit pages): >50
- Unique readers (views): >1,000/month
- Scraper bot uptime: >99%
- Avg. time to find an exam paper: <30 seconds search

---

## 🔄 Development Roadmap

### Phase 1: Foundation (Weeks 1-3)
- [ ] Rent VPS, install MediaWiki + PostgreSQL, configure Cloudflare Tunnel
- [ ] Install VisualEditor + Markdown extension
- [ ] Set up basic structure: Home, Course Index, Semester templates, Forum
- [ ] Deploy scraper bot skeleton (cron job + logging)

### Phase 2: Content Engine (Weeks 4-6)
- [ ] Scraper bot parses university timetable & exam schedule pages
- [ ] Auto-generate course pages with template info
- [ ] File upload system for lecture notes & past exams
- [ ] Seed initial content with 50+ course pages from public sources

### Phase 3: Community (Weeks 7-8)
- [ ] Forum/category system active
- [ ] User onboarding (friendly "edit your first page" tutorial)
- [ ] Notification alerts for bot-detected changes
- [ ] Share with first 50 students, gather feedback

### Phase 4: Polish (Weeks 9-12)
- [ ] Search improvements
- [ ] Mobile theme optimization
- [ ] Documentation & contributor guide
- [ ] Pitch to student council for sponsorship

---

## ⚠️ Risks & Challenges

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Low initial adoption | High | High | Seed with existing open-source course notes; partner with student council |
| Scraper breaks when university site redesigns | Medium | Medium | Monitor with alerts; manual fallback; contribute upstream fixes |
| University copyright concerns (exam papers) | Medium | Medium | Host links, not copies; takedown policy; DMCA-safe |
| Content decay (stale pages) | Medium | Medium | Version history; "last reviewed" tags; community editing culture |
| Bot/VPS downtime | Low | Medium | Healthchecks + automated restart; low-cost VPS with good uptime |
| Student turnover (graduation) | High | Medium | Onboarding new editors each year; wiki is the memory |

---

## 💻 Tech Stack

| Layer | Choice | Why |
|-------|--------|-----|
| Wiki engine | MediaWiki 1.42 LTS | Battle-tested, same as Wikipedia, huge extension ecosystem |
| Markdown | Pagedown extension | Familiar to most students |
| Web server | Nginx | Lightweight, fast, well-known |
| Runtime | PHP 8.2 + PHP-FPM | MediaWiki native |
| Database | PostgreSQL 15 | Reliable, easy backups, supports JSON |
| Scraper | Python 3.11 + BeautifulSoup + Playwright | Easy to write, handles JS-rendered pages |
| Edge | Cloudflare Tunnel | No open ports, free SSL, CDN included |
| VPS | Hetzner CPX21 (~€4/mo) | Cheap EU hosting, fast NVMe, 99.9% uptime |
| File storage | Local disk → MinIO → B2 (gradual) | Start simple, scale as needed |
| Monitoring | UptimeRobot + Healthchecks.io | Free, sufficient for MVP |
| Backups | BorgBackup → Hetzner Storage Box | Encrypted, incremental, cheap |

---

## 📁 Related Files
- `diagrams/architecture.drawio` — System architecture diagram

---

*Last updated: 2026-08-28*

## 📝 Key Features Summary

This is a **student-run, free, ad-free knowledge commons** that provides:

- 🔍 **Centralized academic information** - Everything from timetables to thesis templates
- 🤖 **Automated scraping bot** - Keeps content current from university pages
- 📝 **Markdown editing** - Easy for students to contribute
- 📚 **Media library** - Lecture notes, recordings, past exams
- 👥 **Community forums** - Discussion and Q&A
- 🎯 **Lifecycle coverage** - From freshman to PhD
- 💰 **<€10/month** hosting cost
- 🔐 **Cloudflare HTTPS** - Secure, free SSL
- 🔄 **Version control** - Every edit tracked and reversible
- 🏗️ **Simple architecture** - Easy to maintain and scale

---

*Concept created: 2026-08-28*