# kryard (Rust) — planned

The Rust server-side SDK for Kryard's Turnkey-compatible signing API + EIP-7702 relay.
**Not yet implemented** — this directory reserves the Phase-A slot.

## Plan

- **Generated** typed client + models from [`../../spec/openapi.yaml`](../../spec).
- **Hand-written core**:
  - `X-Stamp` stamper — P-256 signature over `SHA-256(canonical_json(body))` (`p256`/`ecdsa`).
  - canonical JSON.
  - HPKE export decrypt (RFC 9180, suite `DHKEM(P-256,HKDF-SHA256)/HKDF-SHA256/AES-256-GCM`)
    — e.g. the `hpke` crate.
  - relay / EIP-7702 helpers (`alloy`).
- **Tests** assert against [`../../vectors/`](../../vectors).

Publishes as `kryard` on crates.io. Async on `reqwest`/`tokio`.
