# Kryard SDKs

Per-language client SDKs for **Kryard** — the Turnkey-compatible signing API and the
managed EIP-7702 relay. This is a **monorepo**: every language SDK lives here, is
driven by one API spec, and is verified against one shared set of conformance
vectors, so they never drift.

```
sdk/
├── spec/          OpenAPI — the single source of truth for the API surface
├── vectors/       Language-agnostic conformance vectors (every SDK must pass these)
└── packages/
    ├── typescript/   @kryard/sdk        — server + relay + signing + export (isomorphic)
    ├── python/       kryard             — (planned)
    ├── go/           kryard-go          — (planned)
    ├── rust/         kryard             — (planned)
    ├── react/        @kryard/react      — (future: embedded wallets)
    ├── react-native/                    — (future)
    ├── swift/  kotlin/  flutter/        — (future: native embedded wallets)
```

## Two SDK surfaces

Kryard, like Turnkey, has two surfaces — and they map onto two SDK tiers:

| Tier | Auth | What it does | Status |
| --- | --- | --- | --- |
| **Server-side** | API key + `X-Stamp` | create keys, sign, export, drive the relay — from a backend | **shipping** |
| **Client-side** | passkey / OAuth / sessions | embedded wallets in web & mobile | **gated** on the embedded-wallet/auth product |

The current SDKs are **server-side / isomorphic** (the TypeScript package runs in Node
*and* in the browser for the relay's user-signing flow). The client-side embedded-wallet
kits (React, React Native, Swift, Kotlin, Flutter) land once Kryard ships sub-orgs +
passkey/OAuth auth.

## Language coverage

**Phase A — server-side (now):**

| Language | Package | Status |
| --- | --- | --- |
| TypeScript | `@kryard/sdk` (`packages/typescript`) | ✅ available |
| Python | `kryard` (`packages/python`) | planned |
| Go | `packages/go` | planned |
| Rust | `kryard` (`packages/rust`) | planned |

**Phase B — client-side, after embedded-wallet auth ships:** React → React Native →
Swift → Kotlin → Flutter.

## How a language is added

1. The API surface is defined once in [`spec/openapi.yaml`](spec/). The typed client +
   models are **code-generated** from it per language (generator TBD — see
   [`spec/README.md`](spec/README.md)).
2. A thin **hand-written core** sits on top of the generated client — the part that
   can't be generated: the `X-Stamp` stamper, canonical-JSON, the HPKE export decrypt,
   and the relay/EIP-7702 helpers.
3. Every language's tests assert against the shared [`vectors/`](vectors/) — so the Rust
   SDK derives the same Bitcoin address and produces the same X-Stamp as the TypeScript
   one.

So adding a language ≈ generate the client + port the small core + point the tests at
`vectors/`.

## Packages

- **[`packages/typescript`](packages/typescript)** — `@kryard/sdk`. Today: relay
  (`sponsorExecute`/`sponsorCall`), wallets & signing (`createPrivateKey`,
  `signRawPayload`, `signTransaction`), HPKE key export, and the `X-Stamp` stamper.

## License

[MIT](LICENSE).
