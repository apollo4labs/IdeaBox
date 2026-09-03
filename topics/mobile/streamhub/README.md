# 🎬 StreamHub

> One app to browse, search, and launch anything across every streaming service — a unified, subscription-aware catalog with cross-provider watchlists and one-tap deep links that open titles directly inside Netflix, Disney+, Max, Prime Video, or Apple TV+.

**Status:** Concept

---

## One-Liner
A cross-platform (iOS + Android) streaming aggregator that unifies fragmented catalogs into one searchable feed: browse and manage watchlists without caring which provider holds the rights, filter by the services you actually pay for, and tap "Watch Now" to launch the title inside the provider's native app via OS-level deep linking.

---

## 📋 Overview
Subscription fatigue + decision paralysis. The average user juggles 3–5 streaming services, each with its own app, its own search, and its own watchlist. StreamHub sits *on top* of all of them:

- **Unified catalog** — one search and discovery surface powered by TMDB (free)
- **Cross-provider watchlists** — Netflix shows and Max movies live in one list
- **Subscription-aware filtering** — onboarding picks your active services; everything else is filtered out
- **One-tap handoff** — native deep links (`nflx://`, etc.) launch the title in the provider's app, bypassing browser "Open in app?" prompts
- **Cache-first availability** — Supabase Postgres + Edge Functions serve availability with a 24h TTL, hitting paid APIs only on cache miss
- **$0/month infrastructure** — Supabase Free Tier + TMDB free + on-demand availability APIs

---

## ❓ Problem
Content is everywhere, but finding it is harder than ever.

### Pain Points
- **Subscription fatigue:** users maintain 3–5 subscriptions and can't remember which service has which title
- **Decision paralysis:** checking "where can I watch X?" means opening 5 different apps
- **Fragmented watchlists:** no carry-over between services — "My List" on Netflix is useless for Prime content
- **Broken handoffs:** web-based aggregators (JustWatch, Reelgood) open browser prompts ("Open in Netflix?") that kill the flow, or don't launch the native app at all
- **Rights churn:** titles vanish from services without notice; watchlists silently rot

### Current Alternatives
| Alternative | Gap |
|-------------|-----|
| JustWatch / Reelgood | Web-first, browse-only; handoff is a browser redirect, not native launch; no subscription toggle filtering as the core UX |
| Trakt | Tracking/ratings focused, not a launcher; no one-tap playback |
| Provider apps (Netflix etc.) | Only see one catalog; every service is a separate silo |
| Google search "watch X" | Generic results, no cross-app watchlist, no native handoff |

### The Gap
No mainstream app combines a unified catalog, subscription-aware filtering, **and** OS-level one-tap deep-link handoff into the provider's native mobile app — the moment that turns "browsing" into "watching" with zero friction.

---

## 💡 Solution
### Core Feature 1 — Unified Catalog & Search
One search and discovery surface (Trending, Top Rated, Genres) powered by TMDB's free API. Every title resolves to whichever provider currently holds the rights in the user's region.

### Core Feature 2 — Subscription-Aware Filtering
Onboarding wizard: "Only show content on my active Netflix & Max plans." Stored in the user profile (or on-device pre-auth), filtering out services the user doesn't pay for.

### Core Feature 3 — Cross-Provider Watchlists
One watchlist spanning every service. Add from any title page, synced across devices once auth lands (Phase 4).

### Core Feature 4 — One-Tap Deep-Link Handoff
Native OS link handlers (`url_launcher` in Flutter, Android Intents / iOS `UIApplication` calls) launch the provider app directly — no browser prompts. Graceful fallback: if the app isn't installed, open the provider's mobile web portal.

### Core Feature 5 — Cache-First Availability Layer
Supabase Edge Function routes availability lookups: query Postgres cache first (24h TTL), call Watchmode / Streaming Availability API only on cache miss. Keeps paid API usage minimal and responses fast.

### Core Feature 6 — Local Performance
Heavy poster assets and static metadata cached on-device, keeping network requests light.

---

## 👥 Target Users

| Segment | Description | Size |
|---------|-------------|------|
| Primary | Cord-cutters with 3+ active subscriptions who hate checking 5 apps | Large |
| Secondary | Rotators who churn subscriptions monthly and always ask "what's still worth it?" | Large |
| Tertiary | Families sharing one set of services, wanting a single shared watchlist | Medium |

### User Personas
**Persona 1 — "The Multi-Service Binger":** pays for Netflix, Max, and Disney+; starts every evening with "what do we watch?" and gives up after checking three apps.

**Persona 2 — "The Churner":** rotates subscriptions monthly based on what's releasing; needs at-a-glance answers about where things are available *right now*.

**Persona 3 — "The Family Curator":** manages one household watchlist; wants a single list the whole family can add to, regardless of provider.

---

## 🏗️ Architecture

### High-Level Architecture
```
┌────────────────────────────────────────────────────┐
│              Mobile App (Flutter / CMP)            │
│  Discovery Rails │ Search │ Watchlists │ Deep Links│
└───────┬──────────────────────────────┬─────────────┘
        │ REST (metadata, watchlists)  │ deep link launch
┌───────▼──────────────────────────────▼─────────────┐
│          Supabase (Free Tier)                       │
│  Postgres: users, user_watchlists,                  │
│            subscription_preferences,                │
│            availability_cache                       │
│  Edge Functions: availability proxy (cache-first)   │
└───────┬──────────────────────────────┬─────────────┘
        │ cache miss (on-demand only)  │ metadata
┌───────▼──────────────┐   ┌───────────▼─────────────┐
│ Watchmode /          │   │ TMDB API (free)          │
│ Streaming Avail. API │   │ search, details, posters,│
│ region availability, │   │ trailers, cast           │
│ deep links           │   └─────────────────────────┘
└──────────────────────┘
```

### Availability Lookup Flow (cache-first, 24h TTL)
```
1. User requests title availability in their region
2. Edge Function queries Postgres availability_cache for (title, region, service)
3a. Hit (≤24h old) → return cached result, zero external calls
3b. Miss → call Watchmode / Streaming Availability API → store in cache → return
4. App shows "Watch Now" on the owning provider, filtered by user's subscriptions
5. Tap → OS deep link launches provider app (fallback: mobile web portal)
```

### Tech Stack
- **Frontend:** Flutter or Compose Multiplatform (native iOS/Android; native link handling bypasses browser prompts)
- **Backend:** Supabase Free Tier (Postgres, 500 MB)
- **Edge Logic:** Supabase Edge Functions (Deno/TypeScript) — availability proxy + cache invalidation
- **Metadata:** TMDB API (free) — search, details, posters, trailers, cast
- **Availability:** Watchmode or Streaming Availability API — strictly on-demand, region-aware, deep-link resolution
- **Deep Links:** `url_launcher` (Flutter) / Android Intents / iOS `UIApplication`; provider URI schemes (`nflx://`) + Universal/App Links
- **Health/Keep-alive:** GitHub Actions cron pinging the free Supabase DB (free-tier DB pauses after ~7 days of inactivity)
- **Distribution:** TestFlight (iOS) + Internal Testing (Android)

---

## 💰 Business Model
*Not defined in the source plan — candidate streams below, TBD.*

### Revenue Streams (options)
- **Premium tier:** watch history, availability-change alerts, multi-profile/shared lists, advanced filters
- **Discovery sponsorship:** surfaced titles (clearly labeled) from partners
- **Referral/affiliate:** platform sign-up links (weak — majors rarely pay affiliate on subscriptions; verify)

### Unit Economics
- CAC: organic/ASO-first at launch
- LTV: TBD
- Target margin: TBD — goal is $0/month infra overhead at MVP scale

---

## 🚀 MVP Scope

### Must-Have (MVP)
- [ ] Onboarding wizard: pick active streaming services (subscription toggles)
- [ ] TMDB-powered search + discovery rails (Trending, Top Rated, Genres)
- [ ] Cross-provider watchlist (add/remove/status)
- [ ] Availability lookup via Supabase Edge Function (cache-first, 24h TTL)
- [ ] Deep-link resolution for Netflix, Disney+, Max, Prime Video, Apple TV+
- [ ] Native launch handlers + mobile-web fallback when app not installed
- [ ] Supabase schema: `users`, `user_watchlists`, `subscription_preferences`, `availability_cache`

### Should-Have (Post-MVP)
- [ ] Auth + cross-device watchlist/history sync
- [ ] "No longer available" states for titles that leave a service
- [ ] Watch history + "continue watching"
- [ ] Beta programs: TestFlight + Android Internal Testing

### Nice-to-Have
- [ ] Availability-change notifications
- [ ] Price/plan comparison per service
- [ ] Family/shared watchlists
- [ ] Keep-alive + health dashboard (GitHub Actions)

---

## 📈 Success Metrics

### North Star Metric
**Titles launched per active user per week** — the moment the app turns browsing into watching.

### KPIs
- **Operational overhead:** $0/month backend costs (hard constraint, not a growth metric)
- **Availability cache hit rate:** >90% (minimizes paid API calls)
- **Deep-link success rate:** % of "Watch Now" taps that land in the provider app (target >95% with fallback)
- **Handoff time:** tap → provider app foregrounded (target <1s)
- **Watchlist engagement:** average watchlist size per user; % of watchlist items with a "Watch Now" available
- **Activation:** % of users who complete the onboarding subscription picker

---

## 🔄 Development Roadmap

### Phase 1: Foundation & API Proxy Infrastructure
- [ ] Register TMDB + Watchmode / Streaming Availability API accounts
- [ ] Deploy Supabase free instance; schemas for `users`, `user_watchlists`, `subscription_preferences`, `availability_cache`
- [ ] Build & deploy Edge Function: cache-first availability routing with 24h invalidation

### Phase 2: Mobile App Core & Metadata Integration
- [ ] Scaffold Flutter or Compose Multiplatform project
- [ ] Onboarding wizard (active service selection)
- [ ] Home discovery rails + search powered by TMDB

### Phase 3: One-Tap Deep-Link Handoff
- [ ] Deep-link resolution mapping for the 5 major platforms
- [ ] Native OS launch handlers ("Watch Now")
- [ ] Graceful fallback to mobile web portal when app missing

### Phase 4: Personalization & System Health
- [ ] Auth to sync watchlists/history across devices
- [ ] GitHub Actions keep-alive pings for the free Supabase DB
- [ ] Soft launch: TestFlight (iOS) + Internal Testing (Android) — verify handoff reliability across devices

---

## ⚠️ Risks & Challenges

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Availability API cost/limits at scale | High | High | Cache-first with 24h TTL; batch lookups; prototype cost model early |
| Rights churn → dead "Watch Now" links | High | High | "No longer available" states, periodic cache revalidation, user notifications |
| Provider URI scheme changes | Medium | Medium | Abstract deep-link layer; fall back to Universal/App Links, then web portal |
| iOS restrictions on launching third-party apps | Medium | Medium | Use documented schemes + universal links; test TestFlight matrix early |
| Supabase free-tier limits (DB pause, edge invocations) | Medium | Medium | GitHub Actions keep-alives; monitor invocation quotas; exit path to paid tier |
| Watchmode free-tier data gaps | Medium | Medium | Dual-source with Streaming Availability API; region accuracy tests |
| App Store review (aggregator + external links) | Low | Medium | Clear value prop, no pirated content, provider attribution |

---

## 📁 Related Files

- `diagrams/architecture.plantuml` — System architecture & cache-first availability flow
- `diagrams/user-flow.plantuml` — Onboarding → browse → one-tap handoff flow

---

*Last updated: 2026-09-03*