# 🔒 Anonymous Decentralized Chat

> A censorship-resistant messenger where every user runs their own node, messages route through Tor, and identity is a cryptographic onion address — no accounts, no central server, no public IP required.

**Status:** Concept

---

## One-Liner
A peer-to-peer anonymous chat network where each user hosts a lightweight homeserver as a Tor onion service. Messages are routed through the Tor network, identities are derived from keypairs, and no central authority can track, block, or take down the system.

---

## 📋 Overview
A response to the growing threat of platform-level surveillance and regulatory takedown — most immediately, European Chat Control 2.0, which would require scan-and-forward capabilities on end-to-end encrypted messengers. This architecture makes such mandates technically unenforceable: there is no server operator who can be compelled, no plaintext to inspect, and no central point to pressure.

Each participant runs a small homeserver binary on their own hardware — a laptop, a Raspberry Pi, or an always-on box. The server binds a Tor onion service, and the resulting `.onion` address becomes the user's public identity. Peers discover each other through an invite-and-key system, exchange messages routed entirely within the Tor network, and gracefully queue messages when a peer is offline.

---

## ❓ Problem
Centralized and federated messengers all share a structural weakness: someone, somewhere, controls a server. That server is a target for legal compulsion, technical takedown, and metadata harvesting.

### Pain Points
- **Regulatory vulnerability:** New European legislation would require messaging providers to break end-to-end encryption or enable client-side scanning. Any company that can be compelled *will* be compelled.
- **Metadata exposure:** Even encrypted messengers leak who talks to whom, when, and for how long. Central servers collect and retain this data.
- **Single point of failure:** A server block, domain seizure, or corporate shutdown instantly takes down the entire user base.
- **Onboarding friction:** Traditional self-hosted options require port forwarding, a static IP, and a public hostname — a barrier that excludes most non-technical users.
- **Accountability mismatch:** Users who just want to talk to friends don't need an account tied to a phone number, email, or real identity.

### Current Alternatives
- **Centralized messengers (Signal, WhatsApp, Telegram):** Convenient but vulnerable to takedown, metadata collection, and regulatory compulsion. Not trustworthy under hostile legislation.
- **Federated messengers (Matrix/Element):** Better distributed, but still relies on identifiable server operators who can be legally pressured and who can see traffic metadata.
- **Peer-to-peer messengers (Briar, SimpleX):** Strong privacy models, but often require direct connections (Wi-Fi, Bluetooth) or complex setup that limits everyday use.

### The Gap
No existing solution combines all of the following: zero-configuration setup (no port forwarding, no public IP), full Tor routing by default, cryptographic self-sovereign identity, and a lightweight footprint that runs on a daily-driver laptop.

---

## 💡 Solution
A messenger where every user is both a client and a server. No central directory, no account system, no phone number.

### Core Feature 1 — Onion-Service Identity
Each user's node binds a Tor v3 onion service. The `.onion` address is mathematically derived from the node's keypair — it is the identity. No registration, no central directory, no way to link an address to a person or a network. A node behind CGNAT, behind a hotel firewall, on mobile data, all work identically: it just needs outbound connectivity to reach the Tor network.

### Core Feature 2 — Invite-and-Key Bootstrap
New peers join through an invite mechanism backed by cryptographic key exchange. An existing user generates an invite token containing their public key and a one-time use marker. The invitee presents this token to their own node, which performs a key agreement with the inviter's node, establishing an authenticated, encrypted channel. This replaces accounts, passwords, and phone numbers with a trust chain that scales without a central authority.

### Core Feature 3 — Offline Message Queue
If a peer's node is offline, messages are queued by the sender's node and retried automatically when the recipient comes online. The system treats offline nodes as a normal state, not an error — no message loss, no delivery failures.

### Core Feature 4 — Tor-Native Routing
Every message is routed exclusively through the Tor network. No plaintext ever touches the public internet. The architecture gains its anonymity guarantees from Tor's layered encryption and onion routing, not from application-layer tricks.

### Core Feature 5 — Lightweight Background Node
The homeserver binary is written in Rust, designed to run in the background alongside normal computer use with negligible resource overhead. Sleep and lid-close events take the node offline cleanly; it resumes when the machine wakes.

---

## 👥 Target Users

| Segment | Description | Size |
|---------|-------------|------|
| Privacy advocates | Users who refuse to accept surveillance as inevitable | Large, growing |
| Journalists and activists | People who need to communicate without traceability in hostile environments | Niche, high-value |
| European users concerned about Chat Control 2.0 | Anyone in the EU who wants a messenger that cannot be compromised by law | Large |
| Technically-inclined hobbyists | People who enjoy self-hosting and understand the value of self-sovereign infrastructure | Medium |

### User Personas
**Persona 1 — "The Privacy-Conscious Friend Group"**
A small group of friends who want to chat without a company watching. They share onion addresses out-of-band, invite each other, and message freely. They are not threat-modeling state adversaries — they just don't want a corporation building a profile on their conversations.

**Persona 2 — "The Journalist in a Restrictive Region"**
Someone who needs to communicate with sources without leaving a metadata trail. The onion address provides plausible deniability and no account to seize. Offline queuing means messages wait safely if the journalist's node goes dark.

**Persona 3 — "The Self-Hosting Enthusiast"**
A user who runs other services at home and sees this as another tool in the stack. They deploy it on a Raspberry Pi or an old laptop, keep it running 24/7, and connect it to their existing Tor setup.

---

## 🏗️ Architecture

A fully peer-to-peer network over Tor. Each node is a self-contained homeserver that handles its own identity, inbound message queue, outbound message delivery, and peer discovery via invite keys.

### High-Level Architecture
```
┌──────────────┐   Tor (TLS + onion routing)   ┌──────────────┐
│   Node A     │ ◄──────────────────────────► │   Node B     │
│  (onion addr │                                │  (onion addr │
│   A.onion)   │                                │   B.onion)   │
│              │  ┌──────────────────────────┐  │              │
│  ┌─────────┐ │  │  Inbound/Outbound Queue  │  │ ┌─────────┐ │
│  │ Rust    │ │  │  - Stores queued messages│  │ │ Rust    │ │
│  │ homesrv │ │  │  - Retries on reconnect  │  │ │ homesrv │ │
│  │ binary  │ │  │  - Peer state tracking   │  │ │ binary  │ │
│  └────┬────┘ │  └──────────────────────────┘  │ └────┬────┘ │
│       │    │                                  │     │      │
│       ▼    │  ┌──────────────────────────┐    │     ▼      │
│  ┌────────┐│  │  Identity / Key Manager  │    │┌────────┐│
│  │ Tor    ││  │  - Keypair (ed25519)     │    ││ Tor    ││
│  │ socks  ││  │  - Onion service binding │    ││ socks  ││
│  └────────┘│  │  - Invite key generation │    │└────────┘│
│            │  └──────────────────────────┘    │            │
└──────────────┘                                 └──────────────┘
```

### Network Flow
```
1. Node A composes a message to Node B
2. Node A resolves B's onion address (derived from B's public key)
3. Node A opens a Tor connection to B's onion service
4. If B is online: message delivered immediately
5. If B is offline: message stored in B's addressable queue at A,
   retry scheduled, notify A's user of pending delivery
6. When B comes online: B connects to A (or A reconnects to B),
   queued messages transfer, queues clear
```

See `diagrams/architecture.plantuml` and `diagrams/user-flow.plantuml` for visual diagrams.

### Tech Stack
- **Runtime:** Rust (async runtime, lightweight memory footprint)
- **Transport:** Tor v3 onion services over SOCKS5
- **Cryptography:** Ed25519 keypairs for identity, X25519 for key exchange, AES-256-GCM for message encryption
- **Invite system:** Signed invite tokens (JWT-like or custom) binding inviter key to invitee public key
- **Storage:** SQLite or sled (embedded) for message queues and peer state
- **Serialization:** MessagePack or similar compact binary format
- **No database server, no external dependencies, no account service**

### Node Requirements
- A machine with outbound internet access (any connection type works — home broadband, CGNAT, mobile data, hotel WiFi)
- No public IP, no port forwarding, no dynamic DNS
- A Rust toolchain to build the binary
- ~50 MB RAM baseline, negligible CPU during idle

---

## 💰 Business Model

This is not a SaaS product — there is no server to bill, no subscription, no company to pay. The software is free and open source.

### Revenue Streams
- **Donations and sponsorships:** Accept contributions from users who value the software and want to support its development.
- **Paid distributions:** Optional compiled binaries or convenience packages (Docker image, systemd unit files, easy install scripts) for users who don't want to build from source.
- **Supporting infrastructure:** If a public directory/announcement service is ever needed (to help peers discover each other beyond direct invites), it could be funded through donations and run by trusted volunteers.

### Pricing Tiers
| Tier | Price | Features |
|------|-------|----------|
| Core software | $0 | Full client and server, open source, self-hosted |
| Convenience build | $5–10/month | Pre-compiled binaries, auto-updates, packaged configs |

### Unit Economics
- CAC: Near-zero — the software markets itself through its properties
- LTV: Minimal — there is no subscription to extract value from
- Margin: N/A by design. The project exists because a functional anonymous chat is a public good, not a profit center.

---

## 🚀 MVP Scope

### Must-Have (MVP)
- [ ] Rust homeserver binary that binds a Tor v3 onion service
- [ ] Keypair generation and onion address derivation
- [ ] Basic invite token generation and validation (one-use invite codes)
- [ ] Message send/receive over Tor between two known peers
- [ ] Offline message queue with retry on reconnection
- [ ] Simple CLI or minimal TUI for sending/receiving messages
- [ ] End-to-end encrypted messages using per-session keys

### Should-Have (Post-MVP)
- [ ] Multiple peer connections (not just one-to-one, but a small mesh)
- [ ] Group chat support with distributed key management
- [ ] Desktop notification integration (system notification on message arrival)
- [ ] Auto-relay: if a direct Tor connection fails, attempt relay through a trusted intermediate node
- [ ] GUI client (e.g., Tauri or native cross-platform)
- [ ] Address book / peer directory with onion addresses

### Nice-to-Have
- [ ] Bridge mode to accept messages from legacy clients via a volunteer relay
- [ ] Plug-in architecture for alternative transport layers (I2P, CJDNS)
- [ ] Message expiration (disappearing messages enforced at the protocol level)
- [ ] Offline message backlog with configurable size limits
- [ ] Cross-platform packaging (Debian/RPM/Homebrew/Docker)

---

## 📈 Success Metrics

### North Star Metric
Number of active, self-hosted nodes with at least one successful message exchange in the past 30 days.

### KPIs
- **Node uptime:** Percentage of nodes online and reachable over a rolling 7-day window (target: >60% for early adopters)
- **Message delivery latency:** Median time from send to delivery for online peers (target: under 30 seconds over Tor)
- **Invite conversion rate:** Percentage of invite tokens that result in a successfully connected peer (target: >80%)
- **Offline queue depth:** Average number of queued messages per offline peer (target: <50)
- **Binary size and memory:** Homeserver binary under 5 MB, RSS under 50 MB at idle

---

## 🔄 Development Roadmap

### Phase 1: Core Node (Weeks 1–6)
- [ ] Set up Rust project with Tor integration (arti or stem-based)
- [ ] Implement Ed25519 keypair generation and v3 onion service binding
- [ ] Build basic send/receive over a known peer address
- [ ] Implement offline message queue with retry logic
- [ ] CLI interface for compose, send, inbox, peers

### Phase 2: Invite and Key Infrastructure (Weeks 7–10)
- [ ] Invite token format design and implementation
- [ ] Key exchange handshake between inviter and invitee nodes
- [ ] First multi-peer mesh support (3–5 nodes)
- [ ] Integration tests: invite flow, message delivery, offline queue

### Phase 3: Polish and Hardening (Weeks 11–16)
- [ ] End-to-end encryption with per-session ratcheting
- [ ] System notification integration
- [ ] Graceful sleep/resume handling (network reconnect on wake)
- [ ] Basic security audit: key storage, memory handling, error paths
- [ ] Documentation and example deployment configs (systemd, Docker)

### Phase 4: Distribution and Community (Weeks 17–24)
- [ ] Pre-built binaries and packaged distributions
- [ ] Public demo node and test network
- [ ] Community feedback loop with early adopters
- [ ] GUI prototype (optional, parallel track)

---

## ⚠️ Risks & Challenges

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Tor network performance and reliability (latency, guard node selection) | High | High | Use Arti (Rust Tor implementation) for native Tor integration; aggressive retry and queuing; allow relay fallbacks |
| Invite system key compromise (stolen invite tokens) | Medium | High | One-use tokens with short expiry; sign tokens with inviter keypair; revoke compromised keys via out-of-band channel |
| Regulatory pressure on users (not the software, but the operators) | High | High | No logs, no metadata retention, no central service; software distributed as open-source binaries; jurisdictional diversity among users |
| Sybil attacks on the mesh (one attacker creating many fake peers) | Medium | Medium | Invite-only bootstrap limits entry; web-of-trust model grows organically; no anonymous joining without an existing peer vouching |
| Usability barrier for non-technical users | High | Medium | Package with easy install scripts; systemd unit files; Docker image; clear documentation |
| Node offline for extended periods (laptop closed, travel) | High | Low | Offline queuing handles this by design; message expiry configurable; no delivery timeout that breaks the system |
| Tor v3 onion service deprecation or protocol change | Low | Medium | Abstract the transport layer; use well-maintained Tor libraries (arti) that track protocol updates |
| Competing solutions with better UX (Signal, etc.) | High | Medium | This project does not compete on convenience — it competes on what others cannot offer: genuine resistance to legal compulsion |

---

## 📁 Related Files

- `diagrams/architecture.plantuml` — Full system architecture diagram
- `diagrams/user-flow.plantuml` — User flow: invite, send, receive, offline queue
- `mvp.md` — Detailed MVP breakdown with task-level estimates
- `tech-stack.md` — Technology evaluation and rationale

---

## 🌐 Why This Exists Now

European Chat Control 2.0 is the proximate catalyst. If passed, it would force messaging providers to break end-to-end encryption or implement client-side scanning — effectively outlawing true privacy in messaging across a market of 450 million people. The legislation sets a precedent that other jurisdictions will follow. This project is not a theoretical exercise: it is a concrete technical answer to a real, imminent regulatory threat.

The architecture makes compliance impossible not by defiance but by design. There is no server to scan, no company to compel, no operator to arrest, and no backdoor to install. Every participant is a peer, every message is onion-routed, every identity is a keypair. The system cannot be made compliant with Chat Control 2.0 because compliance would require breaking the architecture at its foundation.

---

*Last updated: 2026-08-28*
