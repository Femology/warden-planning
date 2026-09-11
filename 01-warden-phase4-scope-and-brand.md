# Warden — Phase 4: Scope, Brand, Repo Structure

## One-paragraph description
Warden is an open-source risk-policy engine for Stellar smart wallets that decides, per transaction, whether a transfer can proceed with just the wallet's normal signature or needs a step-up confirmation — based on amount, recipient trust, and spending velocity. It exists because embedded-wallet apps (BMONI is the concrete example) currently force biometric confirmation on every single action, which drives real user complaints (verification failures blocking login, screen brightness at night, no setting to turn it off) without making the app meaningfully safer — a $2 transfer to a saved recipient gets the same friction as a $2,000 transfer to a brand-new address.

## Why Stellar primitives are load-bearing, not decorative
Soroban smart wallets already support multiple signers with distinct roles inside `__check_auth` (this is the passkey-kit / smart-account model). Warden slots in as a **policy signer** — the risk decision lives on-chain, next to the wallet itself, rather than in a backend service the wallet provider has to trust and operate separately. The reason to build this on Stellar specifically is that the enforcement point (the smart account's auth check) and the policy engine can share the same trust boundary and the same transaction atomicity. That's a real architectural reason, not a "blockchain because fast/cheap" claim.

## Name
**Warden**

## Repo structure
Four repos, matching the multi-repo pattern used by the strongest approved Stellar projects (e.g. Trustless Work ships contract, dApp, back-office, and docs as separate coordinated repos, not one monorepo). This also maximizes Wave program surface area — four genuinely substantive, independently eligible repos — while each stays scoped enough for a solo maintainer to keep in sync:

- **`warden-contract`** — pure Rust/Soroban workspace. Policy storage, rule evaluation, velocity tracking, unit + integration tests. No UI, no backend, no external dependencies beyond soroban-sdk. This is the artifact the Wave program is actually reviewing for Stellar relevance.
- **`warden-sdk`** — TypeScript client library wrapping the contract: builds and validates policy rule objects, encodes/decodes them for on-chain calls, exposes a single clean `evaluate()`/`setPolicy()` surface. This is the real integration point — what an outside fintech would `npm install`, rather than calling the contract raw. It's what turns "a company could adopt this" from a slogan into something installable.
- **`warden-app`** — reference demo application (recommend a web app, Next.js, over Flutter for v1 — fastest for a Wave reviewer or an outside integrator to click through with no install), built **on top of `warden-sdk`**, not calling the contract directly. Shows: setting a policy, a transfer under threshold (allowed, no step-up), a transfer over threshold or to a new recipient (step-up required), and a velocity-triggered case. Using the SDK here dogfoods it and proves it actually works, not just compiles.
- **`warden-monitor`** — a read-only telemetry service + dashboard that listens to `warden-contract`'s on-chain events (`stepup_required`, `evaluation_allowed`, `velocity_exceeded`) and shows an integrator how often step-up is triggering, on which recipients, and how close accounts are running to their velocity limits. This is what makes Warden operable in production rather than a one-off demo — nobody runs a policy engine blind. **Important constraint:** this service is observability only. It cannot influence any allow/deny decision — that stays exclusively in `warden-contract`. If the monitor could feed back into decisions, it would quietly relocate the actual trust boundary off-chain, which undermines the whole reason to build this on Stellar in the first place.

## v1 scope (decided, not left open)
- **Network:** Stellar Testnet only for v1.
- **Contract count:** one contract, `warden-contract`, covering policy storage + evaluation + velocity tracking. Not split further — a second contract would be speculative at this stage.
- **Asset scope:** single reference asset (e.g. testnet USDC) for policy thresholds. Multi-asset policies are a fast-follow, not v1.
- **Portability:** the *policy rule shape* (a small JSON schema — max amount, daily velocity cap, allowlist, new-recipient flag) is documented separately from the Soroban implementation, so a future EVM adapter (relevant if pitching this to an EVM-based wallet provider) is a translation exercise against a stable spec, not a rewrite of the logic.

## Explicitly out of scope for v1 (so nothing speculative creeps into the build)
- No real biometric hardware integration — the demo app simulates the "step-up" confirmation with a second, distinct confirmation step. Wiring to actual Face ID / fingerprint APIs is an app-layer concern for whoever integrates Warden, not something the contract or reference demo needs to solve.
- No production registration of `warden-contract` as a live signer on a passkey-kit smart wallet yet. The exact signer interface for this needs to be verified against passkey-kit's current source/docs before implementation — it is deliberately **not** fabricated in the Phase 5 spec. This is flagged as a verify-before-build step for whoever picks up that piece.
- No decision-making backend — `warden-monitor` is the one backend service in scope, and it is strictly read-only telemetry over on-chain events. `warden-app` still talks to `warden-contract` for every actual allow/deny decision, via `warden-sdk`, via the Soroban RPC. No service anywhere gets a vote in whether a transfer requires step-up.
- No multi-chain enforcement in v1 — only the rule *shape* is portable; enforcement stays Stellar-native until there's a real integration partner asking for an EVM adapter.
- No alerting, auto-tuning, or policy-recommendation logic in `warden-monitor` for v1 — it displays what happened; it does not suggest or apply policy changes. That would be a real, useful v2 feature, but it's speculative for a first release with no production usage data to learn from yet.

## Next
Phase 5 (contract architecture) is in the companion file. Once you've looked at it and it's solid, I'll generate the Phase 6 contract-build system prompt for your coding agent.
