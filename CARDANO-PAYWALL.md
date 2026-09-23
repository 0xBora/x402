# Working notes: Cardano subpath for `@x402/paywall`

Branch-only file. Delete it before proposing anything upstream.

## Goal

Add `./cardano` to `@x402/paywall`, next to `./evm`, `./svm` and
`./avm`, so a person in a browser can pay a `cardano:*` x402 route with
a CIP-30 wallet. Today they get the static fallback page. The
mechanism (`@x402/cardano`) is already upstream (#2537), so machine
clients can pay but browsers can't.

Owner: Bora Oben (Cardano Foundation, developer advocacy). Thomas
Kammerlocher, who led the Cardano x402 implementation, agreed to review
it (2026-09-23). Implementation starts after TOKEN2049 (Oct 6, 2026).

## How other families got in

- **AVM (Algorand):** the paywall landed inside the mechanism PR #1560
  (merged 2026-04-10, reviewed by phdargen, who also reviewed #2537).
  Use it as the model for the code layout.
- **Stellar:** the mechanism was already upstream, so the paywall came
  as its own contribution: issue #2758, then draft PR #3375. Cardano is
  in the same position. Use it as the model for the process.
- A Go AVM paywall handler (#2142) was closed because Go had no AVM
  mechanism. Go and Python have no Cardano mechanism either, so we emit
  their templates and don't wire handlers.

## Prior art to port

The Cardano Developer Portal's `x402-next` template
(`cardano-foundation/developer-portal`, `examples/templates/x402-next`)
has a working browser flow that settled tADA and tUSDM payments on
preprod:
- `lib/x402/cip30.ts`: wallet discovery and the CIP-30 signer, from
  Thomas's demo
- `lib/x402/payFlow.ts`: the payment flow with `PAYMENT-RESPONSE`
  integrity checks
- `components/Paywall.tsx`: the UI

## Scope

In:
- `typescript/packages/http/paywall/src/cardano/`
- the `./cardano` export and tsup entry
- the `build:paywall` step
- `paywallUtils`, `faucetUrls`, `PaywallApp`
- a CODEOWNERS line for `@x402-foundation/cardano`
- a changeset
- tests

Out:
- Python/Go handler wiring
- `masumi` and `script` methods in the browser
- changes to `@x402/cardano`

## Building blocks

- [ ] Handler and export: `supports()` on `cardano:*`, BigInt amounts,
      config injection via `jsonForScript` (mirror `src/avm/index.ts`,
      `src/avm/paywall.ts`)
- [ ] Wallet layer: CIP-30 discovery on `window.cardano`, connect,
      balance, signer bridge to `ClientCardanoSigner` (counterparts:
      `src/avm/algorand/`, `src/evm/browserAdapter.ts`)
- [ ] Chain data for fee building (open question 1)
- [ ] Payment flow: `x402Client` + `ExactCardanoScheme` client, sign,
      retry with `PAYMENT-SIGNATURE`, check `PAYMENT-RESPONSE`
- [ ] React paywall with Cardano-scoped CSS
- [ ] `build.ts`: es2020 prebundle, emitting TS, Python and Go templates
- [ ] Package integration: tsup, `build:paywall`, `paywallUtils`,
      `faucetUrls`, `PaywallApp`, CODEOWNERS
- [ ] Changeset and README rows
- [ ] Tests, and an e2e on preprod (headless and Eternl)

## Open questions

1. **Chain data:** where does the browser get protocol parameters?
   EVM, SVM and AVM all default to a keyless public endpoint with an
   override, so the proposal is Koios by default plus an override in
   `PaywallConfig`. Blockfrost always needs a key.
2. **Bundle size:** how much Evolution SDK adds to the prebuilt
   template.

## Why Cardano differs from the other families

The wallet layer is lighter: CIP-30 is one injected standard, so no
connector library is needed. Building the transaction is heavier: the
Cardano client builds the complete transaction itself (coin selection,
change, min-UTxO, fee from protocol parameters), where EVM signs an
authorization. That is the price of a facilitator that holds no keys
and pays nothing.

## Working rules

- Follow the AI-assisted development rules in `CONTRIBUTING.md`:
  concise, match existing patterns, verify against
  `specs/schemes/exact/scheme_exact_cardano.md`, no hardcoded
  constants.
- Commits: conventional, signed before anything goes upstream, no
  co-author or session trailers. Disclose AI use in the PR description.
- Push every change. A first implementation was lost to a local-only
  clone.
- Nothing goes to x402-foundation/x402 or cardano-foundation/x402
  without Bora's go.

## Resume

Tracking issue: https://github.com/0xBora/x402/issues/1. Working PR
(draft, the progress log): https://github.com/0xBora/x402/pull/2. Read
the PR comments for findings per block before continuing.

```bash
git fetch upstream && git rebase upstream/main
cd typescript && pnpm install --frozen-lockfile
npx turbo run build --filter=@x402/paywall...
pnpm --filter @x402/paywall test
```

Baseline 2026-09-23 on upstream `6fe0d4bf` (Node 24.1.0, pnpm 11.1.1):
- build: 5/5 tasks
- tests: 77/77
- `build:paywall` regenerates the EVM, SVM and AVM templates one line
  different from the committed ones. Understand why before committing
  our own generated template.
