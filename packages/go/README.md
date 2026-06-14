# kryard-go — planned

The Go server-side SDK for Kryard's Turnkey-compatible signing API + EIP-7702 relay.
**Not yet implemented** — this directory reserves the Phase-A slot.

## Plan

- **Generated** typed client + models from [`../../spec/openapi.yaml`](../../spec).
- **Hand-written core**:
  - `X-Stamp` stamper — P-256 signature over `SHA-256(canonical_json(body))`.
  - canonical JSON.
  - HPKE export decrypt (RFC 9180, suite `DHKEM(P-256,HKDF-SHA256)/HKDF-SHA256/AES-256-GCM`)
    — `github.com/cloudflare/circl/hpke`, the same library the signer seals with.
  - relay / EIP-7702 helpers (`go-ethereum` for tx + EIP-7702 authorizations).
- **Tests** assert against [`../../vectors/`](../../vectors).

Consumed via the module path (e.g. `github.com/Kryard/sdk/packages/go`). Go is
infra-native here — the Kryard signer itself is Go, so the crypto primitives line up.
