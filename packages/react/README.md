# @kryard/react — future (Phase B)

Client-side embedded-wallet kit for React. **Not started** — gated on Kryard shipping
the embedded-wallet/auth layer (sub-orgs, passkey/OAuth login, sessions).

When that lands, this sits on top of the TypeScript core (`@kryard/sdk`) and adds
hooks/components for embedded-wallet creation, auth, and in-browser signing — mirroring
Turnkey's `react-wallet-kit`. Build server-side SDKs (TS/Python/Go/Rust) first.
