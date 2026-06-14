# Conformance Vectors

Language-agnostic test vectors that **every** Kryard SDK (TypeScript, Python,
Rust, …) asserts against. Each language's test suite loads these JSON files and
checks its implementation reproduces the expected values **byte-for-byte**. This
is how we keep cross-language behavior in lock-step: address derivation, X-Stamp
construction, canonical-JSON hashing, and the HPKE export suite must agree
across every binding.

> **All vectors are REAL** — copied from existing tests in the L3 (`l3-kryard`)
> signer/api/SDK, never invented. Every address vector cites the specific Go test
> file it came from. Do not edit a value here without updating the upstream test
> it was copied from (and vice-versa).

## Files

| File | What it pins | Upstream source |
|---|---|---|
| `address.json` | 40 `{curve, publicKey, format, address}` vectors covering all supported address formats | `services/signer/internal/address/*_test.go` and `internal/keys/p256_test.go` |
| `x-stamp.json` | X-Stamp header construction (P-256 DER sig over `sha256(body)`, base64url envelope) | `packages/typescript/test/stamper.test.ts` + `src/stamper.ts` |
| `canonical-json.json` | Canonical JSON serialization + SHA-256 (idempotency / policy / stamp hashing) | `services/signer/internal/canonicaljson/canonicaljson_test.go` |
| `hpke.json` | HPKE export suite descriptor (RFC 9180; round-trip tested in-language) | `services/api/src/signerClient.ts` + `packages/typescript/src/export.ts` |

## `address.json` (40 vectors)

Each entry is `{ name, curve, publicKey (hex), format, address, source }`.
Breakdown by curve: **30 secp256k1**, **9 ed25519**, **1 p256**.

Sources (every vector cites its test file inline via the `source` field):

- **Ethereum** (2: compressed + uncompressed input) — `address/registry_test.go`,
  Ganache account #0 (`0x90F8bf6A…c9C1`). The two entries share an address: the
  deriver normalizes either SEC1 encoding.
- **Solana** — `address/registry_test.go`, RFC 8032 all-zero ed25519 seed.
- **P-256 compressed** — `keys/p256_test.go`, scalar=1 → base point G
  (`036b17d1…c296`); the `ADDRESS_FORMAT_COMPRESSED` "address" is the compressed
  pubkey itself.
- **Cosmos / Sei** — `address/tier23_test.go` (cosmospy `test_vector`; Sei shares
  the HASH160 data with a different HRP).
- **TRON** — `address/tier23_test.go` (rust-tron `test_address_from_public`).
- **Dogecoin mainnet/testnet** — `address/tier23_test.go` (bip_utils
  `test_P2PKH`).
- **XRP** — `address/tier23_test.go` (xrpl.js ripple-keypairs fixtures).
- **Bitcoin (20: 5 script types × 4 networks)** — `address/bitcoin_test.go`,
  generator point **G** compressed. Anchored on BIP-173 / BIP-350 / BIP-341
  published vectors. Note: signet and regtest reuse testnet's base58 version
  bytes, so several base58 addresses repeat by design.
- **Sui (3) / Aptos / Stellar (XLM) / TON v3r2,v4r2,v5r1** —
  `address/ed25519chains_test.go` (Mysten/Aptos/Stellar SDK fixtures; TON
  cross-verified between @ton/ton and tonutils-go).
- **Spark mainnet/regtest** — `address/spark_test.go`, Spark SDK identity public
  key `02ccb26b…f132` (BIP-350 bech32m).

## `x-stamp.json`

`X-Stamp = base64url(JSON{ publicKey, scheme, signature })` where `signature` is
a DER-encoded P-256 ECDSA signature over `SHA-256(UTF-8 body)` and `scheme` is
`SIGNATURE_SCHEME_TK_API_P256`. P-256 signing is **deterministic (RFC 6979)**, so
`stampHeaderValue` is reproducible byte-for-byte. Each vector also includes
`bodySha256Hex` so non-RFC-6979 implementations can still verify by checking the
signature against `publicKey` over that digest.

The keypair (`privateKeyHex = 000…001`) is a throwaway test key — **not** a real
credential. Values were regenerated deterministically from `src/stamper.ts`.

## `canonical-json.json`

The canonical form recursively sorts object keys (codepoint ascending), preserves
array order, and serializes compactly (no insignificant whitespace). `sha256hex`
is the SHA-256 of the canonical UTF-8 string. The `policy_evaluated_input_hash`
vector's hash (`578b79d2…550b`) is the frozen `evaluatedInputHash` pinned in the
Go signer's `canonicaljson_test.go` — the TS API and Go signer must agree on it.

## `hpke.json`

Describes the export suite: `KEM_P256_HKDF_SHA256 / HKDF_SHA256 / AES256GCM`,
HPKE `info = "kryard-export-v1"`, `aad = privateKeyId`. HPKE seal output is
non-deterministic (fresh ephemeral KEM key per seal), so there is no static
decrypt vector; conformance is a **round-trip** assertion in each language
(generate recipient keypair → server-seal a known secret → decrypt → compare).
The TS reference round-trip lives in `packages/typescript/test/export.test.ts`.

## Adding / updating vectors

1. Copy the value from the upstream test (cite the file in `source`).
2. Re-validate: `node -e "require('./vectors/address.json')"` (and friends).
3. Wire it into each language's conformance test loader.
