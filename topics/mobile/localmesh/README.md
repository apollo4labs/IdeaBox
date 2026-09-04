# 🌐 LocalMesh

> A peer-to-peer local communication and resource-sharing app for events, communities, and offline-first scenarios.

**Status:** Concept

---

## One-Liner
_A peer-to-peer app that turns any local network — event venue, campus, festival, or disaster area — into a private mesh for messaging, file sharing, and resource exchange, with no internet, no central server, and no account required._

---

## 📋 Overview
LocalMesh is a local-area peer-to-peer network for people in the same physical space. When a group is together — at a concert, conference, campus, protest, or after a blackout — the internet is often the wrong tool: it's slow, congested, monitored, or down entirely. LocalMesh creates an instant private network over Wi-Fi/Bluetooth where users can chat, share files, and trade resources directly device-to-device.

Unlike internet-dependent apps, LocalMesh is **offline-first by design**: it establishes a mesh among co-located devices, routes data across that mesh hop-by-hop, and only reaches the public internet when explicitly wanted. It's the communications backbone for moments when the network stack matters more than the cloud.

---

## ❓ Problem
In dense physical environments, existing tools fail in three ways:

### Pain Points
- **Congested/exhausted networks** — At festivals or conferences, cellular data is slow, expensive, or fully saturated; local Wi-Fi is locked down.
- **No connectivity in emergencies** — After a blackout, storm, or in a remote area, there is no internet at all, yet people in the same room cannot reach each other digitally.
- **Privacy in shared spaces** — Even with connectivity, public messenger chats and file transfers run through third-party servers and leave metadata trails, which is unwanted for event staff, community organizers, or sensitive exchanges.
- **Onboarding friction** — Setting up a private network normally requires technical know-how (IP addressing, provisioning, servers).

### Current Alternatives
- **Wi-Fi hotspots / tethering** — single access point, single owner, fragile, no mesh redundancy.
- **Public messengers** — internet-required, server-mediated, monitored.
- **File sharing over Bluetooth** — point-to-point only, clunky, no mesh routing or directory.
- **Zero-config internet services (Slack/Discord for events)** — great but need connectivity and central servers.

### The Gap
No existing solution gives a **co-located group an instant, zero-config, internet-independent mesh** for chat, files, and resource sharing — with redundant routing and an offline-first model.

---

## 💡 Solution
LocalMesh is a device-to-device mesh that self-organizes over local wireless interfaces.

### Core Feature 1 — Zero-Config Mesh Formation
Devices discover each other automatically via a lightweight discovery protocol (mDNS/BLE beacons) on the same Wi-Fi network or Bluetooth range. The mesh forms automatically; no IPs, no setup, no server. Each device is a node that can relay for others.

### Core Feature 2 — Offline-First Messaging
Chat messages route across the mesh hop-by-hop. If the internet is absent, messaging still works. Messages store-and-forward: if a recipient is temporarily unreachable, relays hold and deliver when a path reopens. Optionally, a device with internet can act as a bridge to sync into the wider world.

### Core Feature 3 — Local Resource Sharing
Share files, photos, links, and even real-world resources ("need a charger / offering a seat / spare meds") within the local radius. A local directory/feed makes discoverability easy without leaving the network.

### Core Feature 4 — Peer-to-Peer Privacy
Data sent over the mesh is end-to-end encrypted between peers (or within a shared group key). Because there is no central server, there is no central log: the metadata footprint is naturally local and ephemeral. Optional ephemeral/self-destructing messages.

### Core Feature 5 — Internet Bridge (opt-in)
When an internet connection exists, a designated node can bridge the local mesh to a sync/relay service — but the mesh remains fully functional without it. Internet is an enhancement, never a dependency.

---

## 👥 Target Users

| Segment | Description | Size |
|---------|-------------|------|
| Event & festival attendees | Crowds in dense, network-limited venues | Large |
| Campus / workplace communities | People in the same building who want local sharing | Large |
| Emergency / disaster response | Teams in areas with no connectivity | Niche, high-value |
| Community organizers & activists | Groups wanting private, offline comms in shared spaces | Medium |
| Offline-first enthusiasts | Self-hosters and mesh networking hobbyists | Small but influential |

### User Personas
**Persona 1 — "The Festival Attendee"**
_Mara, 26, at a multi-day festival where cellular data is unusable. Wants to coordinate with friends, share photos, and find lost-and-found — all locally, without signal. → Instant local chat and photo/recovery feed._

**Persona 2 — "The Event Organizer"**
_Dev, 33, runs a 500-person conference. Needs reliable, private staff comms and attendee Q&A even when venue Wi-Fi buckles. → Deploys staff mesh; attendees auto-join on the shared network._

**Persona 3 — "The Emergency Coordinator"**
_Ali, a volunteer in an area hit by a blackout. No internet, but dozens of people in one building. → LocalMesh is the communications backbone until infrastructure returns._

---

## 🏗️ Architecture

A decentralized mesh with local-first protocols and an optional internet bridge.

### High-Level Architecture
```
 [Device A] ◀─▶ [Device B] ◀─▶ [Device C]     (ad-hoc mesh, hop-by-hop)
      │               │              │
      ▼               ▼              ▼
 [Local Discovery]  [Local Directory]  [Store-and-forward queue]
 (BLE/mDNS)        (cached per node)
      │
      ▼ (optional bridge node with internet)
 [Internet Bridge] ──> [Optional sync relay]

Encryption: E2E per-peer / shared group key. No central server or log.
```

### Key Design Decisions
- **True peer-to-peer:** no daemon/hub required; any node can relay. The mesh degrades gracefully as nodes come and go.
- **Local-first:** all features work without internet. The cloud is only an *optional* bridge.
- **Resource-light:** designed for phones and low-power devices; message routing tuned for dozens-to-hundreds of nodes.

### Tech Stack
- **Core networking:** Wi-Fi Direct / Bluetooth LE + mDNS/DNS-SD discovery; mesh routing overlay (e.g., a lightweight distance-vector or flooding with TTL for MVP)
- **Transport:** QUIC/WebRTC data channels or raw TCP/UDP on the local link
- **Encryption:** libsodium / Noise Protocol framework for E2E
- **Frontend:** Flutter (Android + iOS)
- **Bridge/relay (optional):** lightweight Node/Go service only for internet sync
- **Storage:** local SQLite per node for messages, directory, and pending relays

---

## 💰 Business Model

### Revenue Streams
- **Free core app** — mesh messaging, sharing, and discovery are free and offline-first.
- **LocalMesh Pro (freemium):** larger mesh limits, internet bridge/relay service, advanced ephemeral-message controls, priority support.
- **Event/Enterprise licenses:** pre-configured deployment for events, venues, campuses, or emergency teams with a managed bridge and monitoring.

### Pricing Tiers
| Tier | Price | Features |
|------|-------|----------|
| Free | $0 | Basic mesh, unlimited local chat/files, E2E encryption |
| Pro | $3–5/mo | Internet bridge, larger meshes, ephemeral messages, analytics |
| Event/Enterprise | Custom | Managed deployment, bridge infrastructure, support, SLA |

### Unit Economics
- CAC: ~$0.5–2 (organic + event partnerships)
- LTV: Pro ~$40–60/yr; Enterprise per-event or annual
- Target margin: >80%

---

## 🚀 MVP Scope

### Must-Have (MVP)
- [ ] Auto mesh formation on shared Wi-Fi (discovery via mDNS)
- [ ] Peer chat with E2E encryption
- [ ] Store-and-forward when peers briefly disconnect
- [ ] Local directory / "who's nearby" feed
- [ ] Basic file/photo sharing across mesh
- [ ] Android + iOS clients (Flutter)
- [ ] Fully offline operation

### Should-Have (Post-MVP)
- [ ] Bluetooth-only fallback mesh (no shared Wi-Fi)
- [ ] Internet bridge node + sync relay (opt-in)
- [ ] Ephemeral / self-destructing messages
- [ ] Group channels & channels for events
- [ ] Larger-mesh routing optimizations (hundreds of nodes)

### Nice-to-Have
- [ ] Resource-trading feed ("offering/need" matching)
- [ ] Mesh health / map visualization
- [ ] Desktop/Linux CLI node
- [ ] Localization & accessibility pass

---

## 📈 Success Metrics

### North Star Metric
**Active peer-to-peer messages exchanged per event/active period** — proof the mesh is being used, not just installed.

### KPIs
- Time-to-mesh from app open: <10 seconds
- Mesh formation success rate (≥2 devices): >98%
- Store-and-forward delivery rate: >95%
- Offline-to-online bridge sync success: >99% (no data loss)
- D7 retention among event users: >35%

---

## 🔄 Development Roadmap

### Phase 1: Core Mesh (Weeks 1–5)
- [ ] Flutter scaffold, local discovery (mDNS)
- [ ] Peer-to-peer chat with E2E encryption
- [ ] Basic store-and-forward relay
- [ ] Local directory / nearby feed
- [ ] Android + iOS builds

### Phase 2: Sharing & Robustness (Weeks 6–9)
- [ ] File/photo sharing across mesh
- [ ] Queued relay improvements, multi-hop routing
- [ ] Reconnect/roaming handling (device moves between APs)
- [ ] On-device testing lab (real devices, multiple APs)

### Phase 3: Bridge & Scale (Weeks 10–13)
- [ ] Internet bridge node (opt-in) + sync relay service
- [ ] Bluetooth-only fallback mesh
- [ ] Group channels / event channels
- [ ] Pilot at 1–2 real events; measure + iterate

### Phase 4: Productization (Weeks 14–18)
- [ ] Pro tier + billing
- [ ] Ephemeral messages
- [ ] Event/Enterprise deployment kit + documentation
- [ ] Security audit (encryption, relay, discovery)

---

## ⚠️ Risks & Challenges

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Mesh routing complexity across varied devices/APs | High | High | Start with flooding/TTL for MVP; instrument heavily; test on real hardware |
| Mobile OS Wi-Fi-Direct restrictions | Medium | High | Design around platform limits; Bluetooth fallback path; ship a host-AP mode where possible |
| Store-and-forward storage sprawl on relays | Medium | Medium | Size limits, TTLs, and deferred infinite-relay (rely on internet bridge) |
| Adoption/awareness for a niche utility | High | Medium | Nail the event/emergency beachhead; partner with event hosts; word-of-mouth |
| Security of an open mesh (spoofing, relays) | Medium | High | E2E by default, key pinning, message signing, explicit trust model |

---

## 📁 Related Files

- `diagrams/architecture.plantuml` — Mesh architecture and bridge
- `diagrams/user-flow.plantuml` — Mesh join, messaging, sharing, bridge flows

---

*Last updated: 2026-09-04*