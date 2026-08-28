# Tech Stack — Anonymous Decentralized Chat

A detailed evaluation of the technology choices for the system. Every choice is driven by one of three goals: anonymity, simplicity, or small footprint. Where a tradeoff exists, the document explains the reasoning.

---

## Language: Rust

**Why:**
- Memory safety without garbage collection — critical for software that runs 24/7 and handles untrusted input
- Strong static typing and ecosystem for cryptography (`ring`, `dalek`, `RustCrypto`)
- Native async support via `tokio` matches the I/O-driven nature of a network daemon
- Single binary deployment — no runtime, no interpreter, no dependency hell
- Excellent Tor integration via `arti`, a Rust-native Tor implementation

**Why not:**
- Go: good async, but weaker static guarantees for crypto code
- C/C++: powerful but unsafe for the threat model here
- Python / Node: dynamic typing is risky for security-critical code; deployment complexity higher

---

## Tor Integration: `arti`

**Why:**
- Pure-Rust implementation maintained by the Tor Project
- Embedded use — no separate `tor` process to manage, no control port to secure
- Active development, modern API, on track to become the reference Tor client
- Can act as both a Tor client and an onion-service host in one process
- Removes the need to bundle, fork, or interface with external `tor` binary

**Tradeoff:**
- `arti` is still maturing; some features (e.g., legacy protocols) are not yet implemented. For an onion-service-only messenger, this is fine.

**Alternative considered:** spawn external `tor` and use control port / SOCKS5. Rejected because of complexity (two processes, auth, data dir coordination) and the fact that `arti` solves this.

---

## Identity / Crypto: Ed25519 + X25519 + ChaCha20-Poly1305

**Why Ed25519 for identity:**
- Short keys (32 bytes), short signatures (64 bytes)
- Fast verification
- Standard, audited, widely supported via `ed25519-dalek`
- Onion v3 address derivation uses Ed25519 keys natively

**Why X25519 for key exchange:**
- ECDH key agreement, fast and safe
- Same keypair family as Ed25519 (birationally related)
- Standard choice for modern protocols (TLS 1.3, Signal, etc.)

**Why ChaCha20-Poly1305 for symmetric encryption:**
- AEAD construction (authenticated encryption in one operation)
- Faster than AES on platforms without hardware AES acceleration
- Used in TLS 1.3, WireGuard, and many modern protocols

**Why not AES-256-GCM:**
- Slightly more vulnerable to timing side-channels in software implementations
- ChaCha20 is simpler to implement correctly

**Key storage:**
- Identity keypair: file at `~/.anonchat/identity.key`, mode 0600
- Post-MVP: support hardware-backed keys (TPM, Secure Enclave) where available

---

## Async Runtime: `tokio`

**Why:**
- The system is heavily I/O-bound (Tor connections, queue polling, peer handshakes)
- `tokio` is the most mature async runtime in the Rust ecosystem
- Excellent ecosystem support (Hyper, Tonic, SQLx, rusqlite all work with it)
- Work-stealing scheduler handles the mix of long-lived Tor circuits and bursty message traffic well

**Why not async-std:** smaller ecosystem, less production-proven.

---

## Storage: SQLite via `rusqlite`

**Why:**
- Embedded — no separate database process
- Battle-tested, audited, single-file storage makes backup trivial
- Sufficient performance for the workload (peers, queue, message log)
- FFI overhead is minimal in this design

**Why not sled / redb / LMDB:**
- Sled is less mature than advertised; some performance issues
- LMDB is fast but more complex and has historically had corruption edge cases
- For a small, embedded queue and peer store, SQLite is the boring, safe choice

**Schema (high-level):**
- `peers(pubkey PRIMARY KEY, alias, added_at, last_seen_at)`
- `outbound_queue(id INTEGER PRIMARY KEY, peer_pubkey, envelope BLOB, attempts INT, next_retry_at INT, created_at INT)`
- `inbound_messages(id INTEGER PRIMARY KEY, peer_pubkey, envelope BLOB, received_at INT, read_at INT)`
- `invites(token_hash PRIMARY KEY, inviter_pubkey, used_by_pubkey NULL, expires_at INT)`

---

## Serialization: MessagePack via `rmp-serde`

**Why MessagePack:**
- Compact binary format (vs. JSON's overhead)
- Schema-less (vs. Protocol Buffers' compile-time coupling)
- Fast encode/decode
- Easy to debug (still human-readable-ish, especially with tooling)

**Why not JSON:** verbosity and parsing cost for every envelope.
**Why not Protobuf:** compile-time schema generation is friction for a small protocol.

---

## Wire Protocol (MVP)

A simple, length-prefixed MessagePack stream over TCP (which itself is over Tor's local onion-service listener).

```
[4 bytes: length, big-endian, u32]
[N bytes: MessagePack-encoded Envelope]
```

**Envelope:**
```
{
    "v": 1,                          // protocol version
    "from": [u8; 32],                 // sender's public key
    "to": [u8; 32],                   // recipient's public key
    "ts": u64,                        // unix timestamp
    "nonce": [u8; 12],                // AEAD nonce
    "payload": [u8],                  // encrypted message bytes
    "sig": [u8; 64]                   // signature over the rest
}
```

**Handshake (post-invite connection):**
1. Client sends `Hello { from_pubkey, invite_token }`
2. Server verifies token signature and expiry
3. Server responds with `Ack { server_pubkey, ephemeral_pubkey }`
4. Client derives shared secret via X25519
5. Both sides switch to encrypted mode

---

## Deployment

### Single Binary
- `cargo build --release` produces a single executable
- No runtime, no external libraries beyond libc and a Tor data dir
- Target: <10 MB binary, <50 MB RSS at idle

### systemd Unit
```ini
[Unit]
Description=Anonymous Chat Node
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
ExecStart=/usr/local/bin/anonchat daemon
Restart=on-failure
RestartSec=10
User=anonchat

[Install]
WantedBy=multi-user.target
```

### Docker (optional)
- Single container with the binary
- Persist `~/.anonchat` to a volume
- No exposed ports (Tor is outbound-only)

---

## Why This Stack Is Right

Every choice optimizes for the same property: **the system is small, self-contained, and auditable**. There is no server framework, no API gateway, no message broker, no orchestrator. A single Rust binary talks to Tor, talks to peers, stores state in SQLite, and does its job.

This minimalism is not aesthetic — it is the security property. The smaller the surface area, the smaller the attack surface. Every dependency, every abstraction, every moving part is a place where things can go wrong. The stack has been chosen so that an interested reviewer can read the source, understand the whole thing, and trust it.

---

*Last updated: 2026-08-28*
