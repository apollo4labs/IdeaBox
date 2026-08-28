# 🌆 Living City Live Wallpaper

> A dynamic, context-aware desktop wallpaper: a living medieval city that builds, farms, and evolves through the day — driven by real local time, weather, and seasons — then "burns down" and rebuilds fresh each evening.

**Status:** Concept
**Style reference:** Arc of Icarus (low-poly / voxel isometric world + Sci-Fi / Modern Operational HUD)
**Visual language guide:** See `design-language.md`

---

## 🎯 The Core Idea

A **live wallpaper** showing a thriving medieval city on an isometric grid. It behaves like a medieval city-builder game, but runs passively as a desktop background. The world is always alive:

- 🏠 The city **builds, farms, and lives** on its own.
- ⛅ **Local weather** and **seasons** change how it looks.
- 🗺️ The map is **user-drawn** or **randomly generated**.
- 🔥 The village **"burns down" and resets** fresh each day (configurable hour, e.g. 18:00).
- 🌙 The **color palette matches the time of day** — darker at night, warm at dusk.

### The Daily Phoenix
A toggle daily cycle: the village grows through the day, then at a reset hour a fire sweeps the map tile-by-tile and everything starts fresh — so no two days look the same.

---

## 🏗️ Recommended Architecture

**Julia simulation backend + WebGL (Three.js) frontend**, run as a live wallpaper host.

### Four Layers

1. **Simulation Brain (Julia)** — city logic: agent behavior, building growth, economy, daily reset.
2. **Map & Input** — user-drawn terrain (water/hills/roads) or procedural noise (Perlin/Simplex) seed.
3. **Data Inputs (Context)** — real local time/date + local weather API (e.g. OpenWeatherMap).
4. **Renderer & Aesthetic (WebGL + CSS)** — isometric voxel view, chamfered sci-fi HUD panels.

---

## 🧭 System Flow

```
                    1 Hz WebSocket stream (ws://127.0.0.1:8081)
   ┌───────────────────────┐            JSON state            ┌─────────────────────────┐
   │   JULIA BACKEND       │ ───────────────────────────────▶ │   WEBGL / HUD FRONTEND  │
   │  Agents.jl (ABM/A*)   │                                  │  Three.js iso canvas    │
   │  Weather & Solar API  │                                  │  CSS "Arc of Icarus" HUD│
   │  Daily Phoenix reset  │                                  │  Map drawing overlay    │
   └───────────────────────┘                                  └────────────┬────────────┘
                                                                rendered HTML canvas layer
                                                                      ▼
                                                       ┌──────────────────────────────┐
                                                       │   LINUX HOST ENVIRONMENT     │
                                                       │  systemd → server.jl (Julia) │
                                                       │  linux-wallpaperengine → html│
                                                       └──────────────────────────────┘
```

See `diagrams/architecture.plantuml` for the full component diagram.

---

## Data Flow

1. **Init:** systemd launches Julia daemon → wallpaper host (linux-wallpaperengine/CEF) renders `index.html` on the desktop layer → frontend opens WebSocket to `ws://127.0.0.1:8081`.
2. **1 Hz Macro Tick (Julia):** agents step → villagers reach jobs and enter a **dormant** 15-min work state (zero CPU) → macro economics update → time check triggers `is_burning`.
3. **State Broadcast:** JSON packet `{time, is_burning, season, stats, tiles, agents}`.
4. **Visual Interpolation (Three.js):** 30–60 FPS linear interpolation of agent coords between 1 Hz ticks for smooth motion; directional light rotates with time; HUD updates.

---

## Key Mechanics

| Mechanic | Description |
|----------|-------------|
| Agent Needs & Movement | State vars (health/wealth/energy); move via A* along tile grid |
| Zoning & Upgrades | Land value `V = f(accessibility, density)`; houses auto-upgrade to bigger buildings past wealth threshold |
| Road & Network | Roads spawn to connect houses to resources (water, power, farms) |
| Resource Logistics | Supply lines from production nodes (farms/mines) to housing |
| Daily Phoenix | Time-checked `mass_extinction_event()` at reset hour; buildings deleted, map reset |
| Seasons | Affect farm output (winter = zero food), ground textures, palette |
| Weather | Rain/snow particles, wetness shader, cloud cover, brightness |

### Recommended Tooling Stack
| Component | Library | Role |
|-----------|---------|------|
| Agent & City Logic | `Agents.jl` (Julia) | Thousands of human agents, spatial movement, economy |
| Differential Dynamics | `DifferentialEquations.jl` | Macro resource consumption, population growth, pollution |
| 3D Isometric Renderer | Three.js or GLMakie | Low-poly meshes, voxel houses, grid paths, ortho camera |

---

## Minimal Julia Prototype (`Agents.jl`)

```julia
using Agents, Random

# Land states and human agents
@enum Zone Land = 0 Farm = 1 House = 2 Road = 3
@agent struct Human(GridAgent{2})
    wealth::Float64
    has_home::Bool
end

# World environment
struct CityWorld
    land_value::Matrix{Float64}
    zones::Matrix{Zone}
end

function initialize_simulation(; dim=(50, 50), initial_pop=20)
    space = GridSpaceSingle(dim; periodic=false)
    properties = CityWorld(zeros(dim), fill(Land, dim))
    model = StandardABM(Human, space; properties=properties, agent_step! = human_step!)
    for i in 1:initial_pop
        add_agent_single!(Human, model; wealth=rand(50.0:100.0), has_home=false)
    end
    return model
end

function human_step!(agent, model)
    pos = agent.pos
    model.land_value[pos...] += 0.1  # land value rises where humans stay
    if !agent.has_home && agent.wealth > 30.0 && model.zones[pos...] == Land
        model.zones[pos...] = House
        agent.has_home = true
        agent.wealth -= 30.0
    end
end
```

---

## Design Language (Arc of Icarus)

- **Palette:** Slate blue-gray `#2D3243`, cool muted indigo, white structural text/icons, accent pops (amber = warnings, cyan = active).
- **UI Geometry:** Asymmetric panels, sharp 45° chamfered corners, semi-translucent backdrops.
- **Viewport:** Diminutive isometric grid (30°/45° projections), low-poly/voxel models.
- **Typography/HUD:** sans-serif mono/technical fonts + telemetry overlay (coords, temp, pressure).

See `design-language.md` for full detail.

---

## Survey / Optimization (Power)

Make it behave like a real wallpaper without roasting your PC:

- **Variable FPS / event-driven loop:** idle → 15–30 FPS; only scale to 60 FPS during camera transitions or burn events.
- **`InstancedMesh`** in Three.js: one draw call for all houses/roads/blocks; update instance matrices only when a tile changes.
- **Decouple logic from frames:** sim loop in a **Web Worker**, 1–2 s tick; main thread just interpolates movement.
- **Low-res canvas:** render at 0.75–0.8× DPR with FXAA (isometric low-poly hides lower resolution).
- **Julia daemon:** ~1 Hz `@async` timer ≈ under 0.5% CPU, < 150 MB RAM.

---

## How the City Representation (Visual & Population)

- **Grid size:** 40×40 → 50×50 tiles.
- **40/30/30 rule:** 40% nature/open space, 30% farmland/industry, 30% dense urban core.
- **Tall structures** to the north/top so they don't block villagers.
- **Population sweet spot:** 150–300 visible agents; <50 feels abandoned, >500 too noisy.
- **Job state machine:** walking (interpolated) → working (dormant, zero CPU) → idle. Batch macro resource updates every 30–60 s.

### City Stages by time of day
| Stage | Buildings | Agents | Strategy |
|-------|-----------|--------|----------|
| Morning/Dawn | 10–20 | 10–20 | Low activity, wildlife |
| Midday peak | 60–100 | 150–200 | 70% dormant, 30% walking |
| Evening/Phoenix | reset | 0 | Fire shaders, clear arrays |

---

## Daily Events Timeline

- **06:00** Morning fog, mass farm departure, smoke emitters.
- **12:00** Market Hour rush, peak telemetry.
- **17:00** Tavern gathering, sunset amber light.
- **18:00** 🐦 **Daily Phoenix Fire Event** (configurable hour).
- **22:00** Night patrols, torches, muted blue HUD (`#0F111A`).

---

## Asset Generation (Free, no artist)

Do **not** pay for AI sprite generation. Two fully-free routes:

1. **Procedural 3D code** (recommended): basic Three.js primitives (cubes, cones, cylinders) with modular functions — swap colors in code for seasons/wealth/fire.
2. **Free low-poly GLTF packs:** [Kenney.nl Medieval Town Kit](https://kenney.nl), Quaternius (villager packs), .gltf + `GLTFLoader`.

Building logic in code (much less work than sprites): no asset pipeline, instant dynamic states, automatic day/night lighting.

---

## Julia Performance Notes

Julia = dynamic + JIT-compiled (LLVM). To get C-level speed:
- **Type stability** (don't let return type depend on value).
- **Avoid globals** in hot loops; `const` for fixed ones.
- **Pre-allocate / mutate arrays** (`fill!` instead of `zeros` in loop).
- **Column-major** iteration.
- **Concrete struct field types.**
- Use `@views`, `@inbounds`, `@simd`.
- Benchmark with `@time`, `@btime`, `@code_warntype`.

---

## Deployment (Linux)

- **Step 1:** systemd user service → runs `server.jl` (Julia daemon).
- **Step 2:** host `index.html` via `linux-wallpaperengine` (layer-shell) or Tauri/Electron transparent visual window.
- **Wallpaper Engine (Windows):** structure as standard web project + `project.json` manifest; verify locally in browser; import later.

### Suggested project layout
```
living-city-wallpaper/
├── backend/            # Julia: Project.toml, server.jl, simulation/
│   └── simulation/     #   agents.jl (ABM/A*) + environmental.jl (weather/solar)
├── index.html          # (Wallpaper Engine entry: project.json + index.html)
├── frontend/
```

---

## Example Wallpaper Engine `project.json`

```json
{
  "title": "Arc of Icarus - City Evolution",
  "description": "Dynamic isometric medieval city simulation driven by weather and time.",
  "file": "index.html",
  "type": "web",
  "general": { "supportproperties": true },
  "options": {
    "reset_time": { "text": "Daily City Reset Hour (0-23)", "type": "slider", "min": 0, "max": 23, "value": 18 },
    "city_speed": { "text": "Simulation Speed Multiplier", "type": "slider", "min": 1, "max": 2, "value": 1 }
  }
}
```

---

## Roadmap / Next Steps

- [x] Concept & architecture
- [ ] Julia `Agents.jl` city-growth prototype (roads + land value)
- [ ] WebGL/Three.js isometric render + CSS HUD
- [ ] Time-based lighting & color palette
- [ ] Weather API integration
- [ ] Daily Phoenix reset logic
- [ ] Linux wallpaper host + systemd service
- [ ] Optionally package for Wallpaper Engine
