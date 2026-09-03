# User Stories: Ultra-Fast Arc-Style Ergonomic Browser

_Source spec captured 2026-09-03. Source of truth for the ArcVia Browser idea._

## Feature Epic

As a mobile power user, I want a lightning-fast, thumb-friendly Android browser that combines Arc's reachable UI and space-focused design with Via's sub-second page loads and custom 120Hz spring physics, so that I can browse effortlessly with one hand without bloat or frame drops.

---

## Story 1: One-Handed Bottom Floating Command Bar

**As a** mobile web user holding my phone with one hand,

**I want to** access the URL bar, search input, and tab switcher from a floating bottom pill,

**So that** I don't have to strain my thumb reaching for controls at the top of the screen.

### Acceptance Criteria

**Given** the browser is open on any website,

**When** I view the display,

**Then** the command bar floats at the bottom center of the screen within comfortable thumb range.

**Given** I tap the floating command bar,

**When** the search overlay expands,

**Then** the soft keyboard opens immediately without screen layout flickering or delayed UI passes.

**Given** I am scrolling down a web page,

**When** scrolling down begins,

**Then** the floating pill smoothly hides or minimizes to maximize screen real estate, reappearing immediately upon a slight upward scroll.

---

## Story 2: Spring-Physics Micro-Animations (Replacing Clunky Transitions)

**As a** user who appreciates smooth interface design,

**I want** UI transitions (tab sheets, search overlay expansion, swipe gestures) to use hardware-accelerated spring physics,

**So that** the app feels responsive, fluid, and frame-drop-free at 90Hz/120Hz.

### Acceptance Criteria

**Given** I open or close a tab overlay,

**When** the surface animates,

**Then** the animation uses a stiff spring physics interpolator (dampingRatio ≈ 0.8, stiffness ≈ 400) rather than a default linear fade/slide.

**Given** I drag a sheet with my thumb,

**When** my finger moves across the screen,

**Then** the UI offset tracks 1:1 with touch input velocity without layout recalculation delays.

---

## Story 3: Zero-Latency Request Filtering (Via-Inspired Speed)

**As a** user concerned with battery life and load times,

**I want** ads, trackers, and telemetry scripts blocked before network requests are initiated,

**So that** web pages load up to 3x faster with minimal RAM overhead.

### Acceptance Criteria

**Given** a web page requests external assets (scripts, images, ad tags),

**When** the request reaches the WebViewClient,

**Then** the local URL filter checks the host prior to opening a network socket.

**Given** a host matches the local blocklist,

**When** request interception triggers,

**Then** the browser returns an empty response immediately without waiting for timeouts.

---

## Story 4: Lightweight Card-Based Tab Switcher (Arc Look without Arc Bloat)

**As a** multi-tasking browser user,

**I want** to switch between active open tabs using a clean card overview,

**So that** I can organize my browsing spaces quickly without exhausting device RAM.

### Acceptance Criteria

**Given** I have multiple tabs open,

**When** I open the tab switcher,

**Then** tabs display as lightweight, rounded surface cards arranged in a thumb-accessible carousel.

**Given** I have tabs open in the background,

**When** total background tabs exceed 2 instances,

**Then** the background WebView instances release their heavy memory footprints while preserving scroll coordinates and state in local cache.

---

## Technical Constraints & Quality Attributes

| Metric | Target | Verification Method |
|--------|--------|---------------------|
| App Cold Start Time | < 350 ms | Android Vitals / Profiler |
| FPS Target | Steady 90 Hz / 120 Hz during gestures | Systrace / GPU Rendering Profiler |
| Memory Footprint (Idle) | < 120 MB RAM | Android Studio Memory Profiler |
| APK Binary Size | < 15 MB | Production Release Build Output |