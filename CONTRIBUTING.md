# Contributing to the Kryard SDKs

This is a multi-language monorepo. Two rules keep the SDKs from drifting apart:

1. **The API surface is defined once, in [`spec/`](spec/).** Don't hand-edit per-language
   request/response types — change the spec and regenerate. (Generator choice is tracked
   in [`spec/README.md`](spec/README.md).)
2. **Crypto/wire behavior is pinned by [`vectors/`](vectors/).** Every language's test
   suite loads the shared vectors (addresses, X-Stamp, canonical-JSON, HPKE) and must
   reproduce them exactly. A new address format / curve / activity means: update the
   spec, add a vector, then every SDK is regenerated and re-tested against it.

## Layout

- `packages/<lang>/` — one SDK per language. Each is **generated client + a thin
  hand-written core** (X-Stamp stamper, canonical JSON, HPKE export, relay/EIP-7702
  helpers). Keep the core small and identical in behavior across languages.
- `spec/` — OpenAPI source of truth.
- `vectors/` — language-agnostic conformance fixtures.

## TypeScript (`packages/typescript`)

```bash
npm install                 # workspace install (from repo root)
npm run build               # -> @kryard/sdk
npm run typecheck
npm test                    # vitest, incl. vector assertions
```

## Adding a language

1. Generate the client + models from `spec/openapi.yaml`.
2. Port the core (stamper, canonical JSON, HPKE, relay helpers).
3. Wire its test suite to load `vectors/*.json` and assert.
4. Add a per-package release job publishing to that ecosystem's registry, versioned in
   step with the API.

## Versioning & release

Per-language packages publish on a tag to their registry (npm / PyPI / crates.io / Go
proxy). Keep versions synced to the API/spec version so "which SDK speaks which API" is
trivial.
