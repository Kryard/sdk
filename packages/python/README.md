# kryard (Python) — planned

The Python server-side SDK for Kryard's Turnkey-compatible signing API + EIP-7702 relay.
**Not yet implemented** — this directory reserves the Phase-A slot.

## Plan

- **Generated** typed client + models from [`../../spec/openapi.yaml`](../../spec).
- **Hand-written core** (the part that can't be generated):
  - `X-Stamp` stamper — P-256 signature over `SHA-256(canonical_json(body))`.
  - canonical JSON.
  - HPKE export decrypt (RFC 9180, suite `DHKEM(P-256,HKDF-SHA256)/HKDF-SHA256/AES-256-GCM`).
  - relay / EIP-7702 helpers.
- **Tests** assert against [`../../vectors/`](../../vectors) (addresses, X-Stamp,
  canonical-JSON, HPKE) so the Python output matches every other language byte-for-byte.

Publishes as `kryard` on PyPI. Likely stack: `httpx`, `cryptography` (P-256 + AES-GCM)
or `pyca`/`hpke`, `pydantic` models from the generated types.
