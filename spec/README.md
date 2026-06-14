# Kryard API Spec

`openapi.yaml` is an **OpenAPI 3.1** description of the Kryard Turnkey-compatible
API. **This spec is the source of truth for the SDKs.** Language clients
(TypeScript, Python, Rust) are **code-generated** from it — hand-written client
code should be the thin ergonomic layer on top, not the wire contract itself.

## Source of truth

When the API and a generated client disagree, the spec wins by definition: the
spec is what we generate from. Therefore the spec must track the live API
(`l3-kryard/services/api`). The seed here was written directly from:

- `services/api/src/enums.ts` — enums (`Curve`, `AddressFormat` (all 38),
  `HashFunction`, `ActivityType`, `ActivityStatus`) are copied **verbatim**.
- `services/api/src/submit.ts` — per-activity parameter and `*Result` field names.
- `services/api/src/signerClient.ts` — `SignerReceipt`, `ExportBundle` shapes.
- `services/api/src/app.ts` + `relay/routes.ts` — the route list and relay shapes.
- `apps/app/src/lib/signing.ts`, `client.ts` — the exact request body shape and
  the `activity.result.<type>Result` response nesting.

## What it covers (seed — a solid start, not 100%)

- **Auth:** the `X-Stamp` API-key security scheme (P-256 DER signature over
  `SHA-256(body)`, scheme `SIGNATURE_SCHEME_TK_API_P256`) with a full
  description. Reproduce it with `vectors/x-stamp.json`.
- **Submit endpoints (POST):** `create_private_keys`, `sign_raw_payload`,
  `sign_transaction`, `export_private_key`, `create_wallet`,
  `create_wallet_accounts`. Request body `{ type, timestampMs, organizationId,
  parameters }`; single-nested activity-envelope response.
- **Relay routes (POST):** `submit_transaction`, `get_transaction`,
  `billing_usage`, `token_usage`, `get_balance`.
- **Components/schemas** for every enum and the activity envelope, results, signer
  receipt, HPKE export bundle, and relay records.

Not yet covered (extend as needed): the `authorize_privy_request` activity, the
`/public/v1/query/*` read endpoints (`whoami`, `get_activity`, `list_activities`,
`get_private_key`, wallet queries), and dev-only `/admin/*` bootstrap routes.

## Wire-shape gotchas baked into the spec

- Query/submit endpoints are **POST**, not REST verbs.
- The activity response is **single-nested**: `activity.result.<type>Result`
  (e.g. `activity.result.signTransactionResult`), with `{seconds, nanos}`
  timestamps — see `tools/compat-harness/wire-contract.md` in L3.
- `signTransactionResult.signedTransaction` is hex **without** the `0x` prefix.
- A recognized-but-unimplemented activity type returns a `FAILED` activity at
  HTTP 200; an **unknown** type returns a 400 Turnkey error envelope.

## Code generation

Generator choice is a **placeholder** — TBD. Candidates: `openapi-generator`,
`oapi-codegen` (Go), `openapi-typescript` + a thin client, or a custom generator
that emits per-language SDKs sharing the `vectors/` conformance suite. Whatever we
pick, the generated wire types must satisfy `vectors/` byte-for-byte.

## Validating

```bash
# any OpenAPI 3.1 validator, e.g.
npx @redocly/cli lint spec/openapi.yaml
# or a quick structural parse
node -e "const y=require('yaml');const fs=require('fs');const d=y.parse(fs.readFileSync('spec/openapi.yaml','utf8'));console.log(Object.keys(d.paths).length,'paths',Object.keys(d.components.schemas).length,'schemas')"
```

## Keeping in sync

Any change to `services/api` request/response shapes or `enums.ts` MUST be
mirrored here in the same change set, then SDKs regenerated. The `vectors/`
directory is the cross-language safety net that catches drift.
