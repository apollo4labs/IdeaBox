# MVP Breakdown — Anonymous Decentralized Chat

This document details the minimum viable product (MVP) — the smallest end-to-end system that demonstrates the architecture and proves the value proposition: two users can self-host nodes, exchange messages routed through Tor, with offline resilience.

## Goal
Demonstrate a working two-node system with:
- Self-hosted onion-service identity
- Invite-based bootstrap (no account)
- End-to-end encrypted message exchange
- Offline message queue with retry
- CLI interface

Everything else (multi-peer mesh, GUI, system notifications, relay mode) is post-MVP.

---

## Task Breakdown

### 1. Project Setup
- [ ] Rust workspace setup with `cargo` and version pinning
- [ ] Choose core libraries (see `tech-stack.md`):
  - `arti` for native Tor integration
  - `ed25519-dalek` for keypairs
  - `chacha20poly1305` or `aes-gcm` for symmetric encryption
  - `tokio` for async runtime
  - `rusqlite` for embedded storage
  - `serde` + `rmp-serde` (MessagePack) for serialization
- [ ] Set up CI (cargo test, cargo clippy, cargo fmt)
- [ ] Project license: AGPL-3.0 (copyleft to prevent proprietary forks)

### 2. Identity Layer
- [ ] Generate Ed25519 keypair on first run
- [ ] Persist keypair to `~/.anonchat/identity.key` with restricted permissions (0600)
- [ ] Derive v3 onion service address from keypair using `arti`'s `HsServiceBuilder`
- [ ] Bind onion service on a free local port
- [ ] Display onion address on startup

**Acceptance:** `anonchat init` creates a keypair and binds an onion service, prints the address.

### 3. Invite Token System
- [ ] Token format: `sign(Ed25519, payload)` where `payload = {inviter_pubkey, nonce, expiry, role}`
- [ ] Encode as base64url for shareability
- [ ] CLI: `anonchat invite` produces a one-use token with default 7-day expiry
- [ ] CLI: `anonchat accept <token>` validates, decodes, and prepares to connect
- [ ] One-time-use marker: token includes a random nonce that the inviter tracks; replay rejected

**Acceptance:** Inviter generates token; invitee imports; both sides log a successful exchange.

### 4. Tor Integration
- [ ] Spawn or attach to a Tor daemon (start `tor` process or use embedded `arti`)
- [ ] Configure Tor data directory per-node
- [ ] Expose onion service to the homeserver via a local SOCKS port or direct Tor control
- [ ] Handle Tor startup failure gracefully (retries, clear error messages)

**Acceptance:** Two local nodes can connect to each other over their onion addresses, verified by a ping/test message.

### 5. Message Protocol
- [ ] Wire format: `MessagePack`-encoded envelope
  ```
  Envelope {
      from: PubKey,
      to: PubKey,
      timestamp: u64,
      payload: EncryptedBytes,
      nonce: [u8; 12],
  }
  ```
- [ ] Per-session symmetric key derived from X25519 ECDH between A and B
- [ ] Optional ratcheting for forward secrecy (post-MVP for MVP)
- [ ] Inbound message handler verifies and decrypts

**Acceptance:** Send a message from A to B; B decrypts and displays it. B replies; A decrypts and displays it.

### 6. Offline Queue
- [ ] SQLite schema: `outbound_queue(id, peer_pubkey, envelope, attempts, next_retry_at)`
- [ ] Background tokio task polls queue, retries due messages
- [ ] Exponential backoff: 1s, 5s, 30s, 2m, 10m, 1h
- [ ] On successful delivery, mark row as delivered (configurable retention)
- [ ] Optional: cap queue size, drop oldest, notify user

**Acceptance:** A sends a message while B is offline; queue row created; when B comes online, message delivered automatically.

### 7. CLI Interface
- [ ] `anonchat init` — first-run keypair + onion service setup
- [ ] `anonchat identity` — show current onion address
- [ ] `anonchat invite` — generate invite token
- [ ] `anonchat accept <token>` — accept invite, handshake, add peer
- [ ] `anonchat peers` — list known peers and their status
- [ ] `anonchat send <peer> <message>` — compose and send
- [ ] `anonchat inbox` — read recent messages
- [ ] `anonchat watch` — foreground listener, print new messages as they arrive
- [ ] `anonchat daemon` — run in background (used by systemd)

**Acceptance:** Full invite → accept → send → receive → offline queue cycle completes from CLI.

### 8. Persistence and State
- [ ] SQLite for: peers, sent messages, received messages, outbound queue
- [ ] Default data dir: `~/.anonchat/`
- [ ] Schema migrations handled on startup
- [ ] Backup-friendly: a single file copy restores full state

**Acceptance:** Kill the node, restart it, all peers and queue contents survive.

### 9. Security Hardening (MVP-level)
- [ ] Keypair file permissions: 0600
- [ ] Constant-time comparison for token signatures
- [ ] Rate limiting on inbound connections (defense against scanning)
- [ ] Tor control port auth disabled or password-protected
- [ ] Clear-text logs: never log message contents or onion addresses; rotate logs

**Acceptance:** A security checklist review passes for the MVP scope.

### 10. Packaging and Deployment
- [ ] Dockerfile (single binary + bundled tor)
- [ ] systemd unit file (`anonchat.service` with restart=on-failure)
- [ ] README with quickstart (5-minute setup)
- [ ] Build instructions (`cargo build --release`)
- [ ] Binary size verification (target: <10 MB for the MVP)

**Acceptance:** A fresh user can follow the README and have a working node in under 15 minutes.

---

## Out of Scope for MVP
- GUI client
- Multi-peer groups
- Relay mode
- System notifications
- Forward secrecy ratcheting
- File transfers
- Voice/video

These are tracked in the main README under "Should-Have" and "Nice-to-Have."

---

## Definition of Done
The MVP is complete when:
1. Two users on two separate machines can install the binary, generate identity, exchange invites, and send messages.
2. Messages route over Tor and never touch the public internet in cleartext.
3. A node can be offline indefinitely; queued messages deliver on reconnect.
4. No public IP, port forwarding, or dynamic DNS is required at any point.
5. All security checklist items pass.
6. Documentation is sufficient for a third party to deploy and use the system.

---

*Last updated: 2026-08-28*
