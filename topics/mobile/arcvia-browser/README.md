# 🚀 ArcVia Browser

> A lightning-fast, thumb-friendly Android browser that combines Arc's reachable UI and space-focused design with Via's sub-second page loads and custom 120Hz spring physics — effortless one-handed browsing without bloat or frame drops.

**Status:** Concept

---

## One-Liner
An ergonomic Android browser for mobile power users: Arc-style floating bottom command bar and card tab switcher, Via-style zero-latency request filtering, and hardware-accelerated spring-physics animations tuned for 90/120Hz displays — all under 15 MB APK and 120 MB RAM.

---

## 📋 Overview
Two design philosophies, one browser. Arc proves that browsers can feel like native apps — reachable UI, space-focused surfaces, delightful motion. Via proves that a tiny, aggressive local blocker makes the mobile web feel instant. ArcVia combines both:

- **Ergonomics first:** every control lives in a floating bottom pill, within comfortable thumb range
- **Speed by interception:** ads, trackers, and telemetry die before a network socket opens
- **Fluid by physics:** every transition runs on stiff, hardware-accelerated springs (dampingRatio ≈ 0.8, stiffness ≈ 400), not linear fades
- **Lightweight by design:** background tabs shed their WebView memory footprint, idle RAM stays under 120 MB, APK under 15 MB, cold start under 350 ms

---

## ❓ Problem
Mobile browsers are stuck between two failure modes:

### Pain Points
- **Reachability:** URL bars and tab controls sit at the top of the screen, forcing thumb gymnastics on large phones
- **Janky transitions:** default linear fades/slides feel sluggish and drop frames on high-refresh displays
- **Bloat & battery drain:** ad/tracker scripts consume bandwidth, CPU, and RAM on every page load
- **Tab sprawl:** background tabs hold heavy WebView instances, exhausting device memory
- **Clunky feel:** generic Chromium UI with no spring physics, no 1:1 touch tracking, no sense of speed

### Current Alternatives
| Alternative | Gap |
|-------------|-----|
| Arc (mobile) | Beautiful UI but heavy, RAM-hungry, not tuned for sub-second loads |
| Via Browser | Ultra-fast and tiny, but dated UI, no spring physics, no modern tab overview |
| Chrome / Samsung Internet | Reliable but generic: top-mounted controls, stock transitions, no aggressive filtering |
| Kiwi / Firefox | Featureful but bloated memory footprint and stuttery gestures on 120Hz |

### The Gap
No Android browser combines Arc's ergonomic, space-focused interaction design with Via's interception-level speed and true spring-physics motion — while staying lean enough for a <15 MB APK.

---

## 💡 Solution
An Android WebView-based browser with four pillars, each mapped to a user story:

### Core Feature 1 — One-Handed Bottom Floating Command Bar
A floating pill at the bottom center of the screen holds the URL bar, search input, and tab switcher — always within thumb range.
- Tapping the pill expands a search overlay; the soft keyboard opens immediately with no layout flicker or delayed UI passes
- Scrolling down hides/minimizes the pill to maximize screen real estate; a slight upward scroll brings it back instantly

### Core Feature 2 — Spring-Physics Micro-Animations
All UI transitions (tab sheets, search overlay expansion, swipe gestures) use a hardware-accelerated spring interpolator instead of linear fades/slides.
- Stiff spring physics: `dampingRatio ≈ 0.8`, `stiffness ≈ 400`
- Sheets track touch input 1:1 with velocity, no layout recalculation delays — steady 90/120Hz during gestures

### Core Feature 3 — Zero-Latency Request Filtering (Via-Inspired)
Ads, trackers, and telemetry are blocked *before* any network request is initiated.
- Requests are checked against a local URL filter at the `WebViewClient` interception point, prior to opening a network socket
- Blocked hosts get an immediate empty response — no timeouts, no wasted bandwidth, up to 3x faster page loads with minimal RAM overhead

### Core Feature 4 — Lightweight Card-Based Tab Switcher
Tabs are organized as lightweight, rounded surface cards in a thumb-accessible carousel.
- When background tabs exceed 2 instances, their WebView instances release heavy memory footprints while scroll coordinates and state are preserved in local cache
- Arc's visual language without Arc's bloat

---

## 👥 Target Users

| Segment | Description | Size |
|---------|-------------|------|
| Primary | Mobile power users who browse one-handed and hate jank | Large |
| Secondary | Speed/battery-conscious users who want ad-free loads without full ad-blocker setup | Very large |
| Tertiary | Design-sensitive users drawn to Arc's aesthetics but put off by its resource use | Medium |

### User Personas
**Persona 1 — "The One-Handed Commuter":** browses on public transport with one hand, thumb reach is everything, hates top-of-screen controls and stuttery scroll.

**Persona 2 — "The Speed Purist":** ex-Via user who loves sub-second loads and tiny memory footprint but wants a modern, animated UI.

**Persona 3 — "The Arc Fan, Frustrated":** loves Arc's look and feel but won't accept its RAM appetite on a mid-range phone.

---

## 🏗️ Architecture

Android app built around a single WebView engine with a native, physics-driven UI layer.

### High-Level Architecture
```
┌────────────────────────────────────────────────────┐
│                   UI Layer (Compose)               │
│  Floating Bottom Command Bar │ Spring Animations   │
│  Card-Based Tab Switcher     │ 1:1 Gesture Tracking│
└───────────────┬────────────────────────────────────┘
                │
┌───────────────▼────────────────────────────────────┐
│              Browser Core (WebView)                │
│  WebViewClient.onShouldInterceptRequest ──┐        │
└───────────────┬───────────────────────────┼────────┘
                │                           │
┌───────────────▼──────────────────┐  ┌─────▼──────────────┐
│      Local URL Filter            │  │ Tab State Manager  │
│  (blocklist check BEFORE socket) │  │ (>2 bg tabs → drop │
│  → immediate empty response      │  │  WebView, cache    │
└──────────────────────────────────┘  │  scroll+state)     │
                                      └────────────────────┘
```

### Request Filtering Flow
```
1. Page requests external asset (script, image, ad tag)
2. Request reaches WebViewClient interception point
3. Local URL filter checks host against blocklist
4a. Match → return empty response immediately (no socket opened, no timeout)
4b. No match → allow request, open network socket
```

### Tech Stack
- **Frontend/UI:** Jetpack Compose with custom spring physics interpolators (`dampingRatio ≈ 0.8`, `stiffness ≈ 400`), hardware-accelerated layers
- **Browser Engine:** Android WebView / WebViewClient
- **Filtering:** Local host-blocklist URL filter, interception before socket creation
- **Tab Management:** Background WebView lifecycle management (drop heavy instances >2 bg tabs, persist state locally)
- **Performance tooling:** Android Vitals, Systrace / GPU Rendering Profiler, Android Studio Memory Profiler

---

## 🚀 MVP Scope

### Must-Have (MVP)
- [ ] Floating bottom command bar (URL, search, tab switcher) with scroll-hide/show
- [ ] Search overlay that expands immediately with keyboard, no layout flicker
- [ ] Spring-physics animation system (stiff springs on tab sheets & overlay expansion)
- [ ] 1:1 touch tracking for draggable sheets
- [ ] WebViewClient request interception with local host blocklist → instant empty responses
- [ ] Card-based tab switcher carousel
- [ ] Background tab memory shedding (>2 tabs) with state/scroll restoration
- [ ] Hit quality targets: cold start <350 ms, idle RAM <120 MB, APK <15 MB

### Should-Have (Post-MVP)
- [ ] Custom filter list management (import/export, user rules)
- [ ] Reader mode with spring-transition entrance
- [ ] Password manager / autofill integration
- [ ] Private/incognito mode with isolated WebView profile

### Nice-to-Have
- [ ] Arc-style Spaces / profiles
- [ ] Gesture customization (swipe zones, pill actions)
- [ ] Split-screen adaptive bottom bar
- [ ] Boomerang/peek tab previews

---

## 📈 Success Metrics

### North Star Metric
Median page-load time (with filtering active) — the speed users can feel.

### KPIs
- **App cold start time:** <350 ms (Android Vitals / Profiler)
- **FPS during gestures:** steady 90Hz/120Hz (Systrace / GPU Rendering Profiler)
- **Memory footprint (idle):** <120 MB RAM (Android Studio Memory Profiler)
- **APK binary size:** <15 MB (release build output)
- **Blocked request ratio:** % of intercepted requests returning empty responses without timeout
- **Tab restore fidelity:** % of background tabs restored to exact scroll position

---

## 🔄 Development Roadmap

### Phase 1: Skeleton & Speed (Weeks 1–4)
- [ ] Project scaffold: Compose + WebView, minimal single-tab browser
- [ ] Floating bottom command bar with scroll-hide/show
- [ ] WebViewClient request interception + local blocklist (first 10k hosts)
- [ ] Baseline cold-start and RAM measurements against targets

### Phase 2: Motion (Weeks 5–8)
- [ ] Spring physics engine (dampingRatio ≈ 0.8, stiffness ≈ 400)
- [ ] Search overlay expansion with immediate keyboard focus
- [ ] Sheet drag with 1:1 touch tracking
- [ ] 90/120Hz profiling pass (Systrace)

### Phase 3: Tabs & Polish (Weeks 9–12)
- [ ] Card-based tab carousel
- [ ] Background WebView shedding with state caching (>2 tabs)
- [ ] Memory profiling pass (target <120 MB idle)
- [ ] Release build size optimization (target <15 MB APK)

---

## ⚠️ Risks & Challenges

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| WebView fragmentation across OEMs/devices | High | High | Test against major devices; feature-detect; fallback animations |
| Blocklist false positives breaking sites | Medium | High | Curated list, per-site exceptions, reporting UI |
| 120Hz jank from WebView compositing | Medium | Medium | Hardware-accelerated layers, offload animations off the WebView surface |
| Tab state cache correctness after WebView teardown | Medium | Medium | Persist scroll + DOM state, test restore fidelity |
| Android 15+ WebView policy changes | Low | Medium | Track WebView updates, abstract engine behind an interface |

---

## 📁 Related Files

- `diagrams/architecture.plantuml` — System architecture & request filtering flow diagram
- `user-stories.md` — Original user stories spec (epic, 4 stories, acceptance criteria, quality targets)

---

*Last updated: 2026-09-03*