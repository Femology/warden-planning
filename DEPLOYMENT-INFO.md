# Warden — Deployment & Environment Reference

Single source of truth for every address, credential location, and env var this project
has produced so far. Update this file whenever a new repo is created or a new
environment is deployed — don't let this drift out of sync with reality.

This file itself lives at https://github.com/Femology/warden-planning along with the
master PRD and every phase build prompt — if you're picking this up from a different
machine, clone that repo first and hand this file to a new Claude Code session before
anything else.

---

## Resume here — UI work, step by step

**This section now describes Design System v2** (`13-warden-design-system-v2.md`, this
repo) — the palette pivoted from v1's blue-ink base to a forest-black one, a deliberate
re-skin, not an incremental edit. Anything below dated before this note describing v1
colors is historical.

**Done and verified (real tests, real builds, real production build, checked in an
actual browser against the running dev server — not just written):**
- **Step 1 of the v2 redesign shipped in `warden-app`**: the full v2 token set (dark +
  light, see `13-warden-design-system-v2.md` §2) wired into `globals.css` with a
  runtime-switchable `[data-theme]` mechanism (system preference by default, an
  explicit toggle persisted to `localStorage`, a blocking inline script in
  `layout.tsx` so there's no flash of the wrong theme on load). Fonts (Bricolage
  Grotesque / Instrument Sans / JetBrains Mono) unchanged and confirmed still correct.
  A new persistent, glassmorphism `SiteHeader` (logo mark + wordmark, four marketing
  nav links, theme toggle, a compact Freighter-only connect button) now renders from
  the root layout on every route.
- Freighter wallet connect works (`@creit.tech/stellar-wallets-kit`), passkey kept as a
  secondary "beta" option (`ConnectWallet.tsx`, currently unused by any page since the
  new header uses its own compact `HeaderConnectButton` instead — still there for
  whatever dedicated connect surface wants the fuller passkey flow).
- Phase 18's "Explain this" feature is fully built (see its own section below) and
  already themed correctly under v2 — `StepUpConfirmModal` and `warden-monitor`'s
  `WalletDrilldown` both render it.

**Not done yet — the real next-session list, per the design system's own recommended
build order (§ "Full sitemap" / the doc's own "recommended build order" note):**
1. **The four marketing nav links 404.** `/how-it-works`, `/security`, `/developers`,
   `/protocol` don't exist yet — the header links to their real eventual routes on
   purpose (not a placeholder href), but none of those pages are built.
2. **The landing page itself is still the old v1 single-section demo** (the drag-slider
   hero + three-signals grid), just re-skinned in v2 colors — not yet the full spec
   from `13-warden-design-system-v2.md` §5 (the Interactive Sandbox Terminal with
   1-click stories, the $12-coffee-vs-$5,000-drain comparison, the 5-state security
   model visual, the open-source repo carousel).
3. **The v1 illustrations/logo need a real re-skin check, not just left as-is.** They
   were hand-colored for v1's palette (jade/amber/purple on blue-ink) — confirm they
   still read correctly against the new forest-black base before assuming they do.
4. `/policy`, `/transfer`, `/velocity` still use the old plain layout (now under the
   new persistent header, but no other v2 treatment) — and per the new sitemap these
   should eventually move under `/app/...` (e.g. `/app/policy`), not stay at root;
   that's a real routing decision to make deliberately, not drift into.
5. No guardian/recovery, flagged-address, or account-state UI anywhere yet — the SDK
   supports all of it (v0.3.0+), nothing in either app's UI calls any of it.
6. GSAP + Lenis motion, the Paper Shaders hero effect, and `warden-monitor`'s dashboard
   getting any of this pass at all are all still fully unstarted.

**To pick this back up:** open a Claude Code session pointed at this file, say
"continue the v2 UI work from `DEPLOYMENT-INFO.md`," and go through the list above in
order. The user is currently doing hands-on design work in Antigravity — check with
them before starting new UI code, since work may already be in progress there that
hasn't been pushed yet.

---

## Things only you can do, with exact steps

Everything else in this project is done and committed. These specific steps need your
own accounts, your own browser, or a human physically doing something — I can't do any
of these on your behalf.

### 1. See the actual UI — run `warden-app` locally

Already running as of this session (I started it). To restart it yourself later:

```bash
# in WSL
cd ~/projects/warden-app
npm run dev
```

Then open **`http://localhost:3000`** in Chrome or Edge **on Windows** (not inside
WSL) — WSL2 forwards `localhost` automatically. Click **"Create wallet"**: this
triggers a real Windows Hello prompt (whatever you have set up — fingerprint, PIN,
face). That's a genuine WebAuthn passkey ceremony, not a mock. After connecting,
you'll land on `/policy` (no policy exists yet), set one, then try `/transfer`.

If `localhost:3000` doesn't load in your browser: tell me, and check that nothing
else on your PC is already using port 3000.

### 2. Deploy `warden-app` to Vercel

1. Go to [vercel.com](https://vercel.com), **New Project**, import
   `Femology/warden-app` from GitHub.
2. Set every env var from the table in `warden-app`'s own README — copy the real
   values from the "Testnet deployment" section below. **`WARDEN_DEPLOYER_SECRET`
   goes in as a server-only env var** (Vercel's dashboard has a toggle for this) —
   never prefix it with `NEXT_PUBLIC_`.
3. Deploy. Once live, open the URL and walk all three demo scenarios yourself to
   confirm it actually works against Testnet, not just that the build succeeded.
4. Come back and tell me the URL — I'll update this file, `warden-contract`'s release
   notes, and the docs site.

### 3. Deploy `warden-monitor`'s indexer to Render

1. Go to [render.com](https://render.com), **New +** → **Blueprint**, connect
   `Femology/warden-monitor`. It will detect `render.yaml` automatically.
2. Render will prompt you for the env vars marked `sync: false` in that file
   (`WARDEN_CONTRACT_ID`, `WARDEN_RPC_URL`, `WARDEN_NETWORK_PASSPHRASE`,
   `WARDEN_DEPLOY_LEDGER`) — copy the real values from below.
3. **Confirm the persistent disk actually works**: after the first deploy, trigger a
   manual redeploy and check `/summary` still shows the same event counts as before.
   If they reset to zero, the disk isn't mounted correctly — tell me and I'll debug
   the config.

### 4. Deploy `warden-monitor`'s dashboard to Vercel

Same as step 2, but import `Femology/warden-monitor` with **Root Directory set to
`dashboard`**, and set `NEXT_PUBLIC_INDEXER_URL` to the Render URL from step 3.

### 5. Connect `warden-docs` to GitBook

1. Go to [app.gitbook.com](https://app.gitbook.com), create a space, and use its
   **Git Sync** feature to connect `Femology/warden-docs`.
2. It reads `.gitbook.yaml`/`README.md`/`SUMMARY.md` automatically — no build step.
3. Every future doc change: just push to `main` in that repo, GitBook picks it up.

### 6. Record the demo video

Once steps 2–4 are live: screen-record yourself walking through all three scenarios
in the deployed `warden-app` (Allow, step-up by amount or new recipient, step-up by
velocity) — a few minutes, real clicks, real Windows Hello prompts. This is the one
thing here that's inherently manual; I can't operate a browser as a human.

### 7. The actual Wave submission

Once the above links are real: go to
[drips.network/wave/stellar](https://www.drips.network/wave/stellar), apply with
`warden-contract` (repo applications are capped at 5 per wave; KYC is required). Use
the project description, repo-relationship description, and planned-issues
description already written earlier in this conversation — paste them into the
submission form as-is or adapt them.

### A security note on the deployer key

`WARDEN_DEPLOYER_SECRET` (below) currently sponsors every transaction fee in
`warden-app` and lives in your local `.env.local`. It's a Testnet-only key with no
real value right now, but if you ever point any of this at Mainnet, rotate it and
treat it as a real secret — don't reuse this exact key.

---

## Repos

| Repo | URL | Status |
|---|---|---|
| `warden-contract` | https://github.com/Femology/warden-contract | Deployed to Testnet, Phases 14+15+16 (dual velocity/trust decay, flagged-address registry, guardian recovery). CI + branch protection. 58/58 tests. [v0.3.0](https://github.com/Femology/warden-contract/releases/tag/v0.3.0). `WARDEN-PROTOCOL.md` (protocol v1.0.0) + `CHANGELOG.md` + a protocol-rule-change issue template now govern any future change to a state transition or step-up reason — v0.2.0/v0.1.0 are retired earlier deployments |
| `warden-sdk` | https://github.com/Femology/warden-sdk | 73/73 tests. CI + branch protection. **Use tag `v0.4.0`** — adds the Phase 18 explain module (`buildExplainPrompt`/`validateExplanationResponse`/`fallbackExplanation`) and fixes `AccountState`/`GuardianConfig`/`RecoveryProposal` never being exported from the package's own entry point; earlier tags are pre-Phase-18 or have known bugs (see the version-history sections below) |
| `warden-app` | https://github.com/Femology/warden-app | 24/24 tests. CI + branch protection. On `warden-sdk` v0.4.0. Phase 18 "Explain this" live in `StepUpConfirmModal`. Design System v2 Step 1 shipped (tokens, fonts, persistent nav shell — see "Resume here" above). [v0.1.0](https://github.com/Femology/warden-app/releases/tag/v0.1.0) (release not yet re-tagged). No public URL yet (issue #2) |
| `warden-monitor` | https://github.com/Femology/warden-monitor | 30/30 tests (indexer+dashboard). CI + branch protection. On `warden-sdk` v0.4.0. Phase 18 "Explain this" live in `WalletDrilldown`. [v0.1.0](https://github.com/Femology/warden-monitor/releases/tag/v0.1.0) (release not yet re-tagged). Not deployed yet (issues #2, #3) |
| `warden-docs` | https://github.com/Femology/warden-docs | Complete GitBook site (6 pages), caught up through Phase 18 — contract ID, SDK version, and the evaluate() decision order all current; Phase 15/16's full reference now points to `WARDEN-PROTOCOL.md` rather than duplicating it. Not yet connected to app.gitbook.com — no live URL yet |
| `warden-planning` | https://github.com/Femology/warden-planning | This file, the master PRD, and every phase build prompt. Clone this first on a new machine. |

**All five repos are also cloned locally, right next to this file**, at
`warden-contract/`, `warden-sdk/`, `warden-app/`, `warden-monitor/`, `warden-docs/`
(siblings of this `DEPLOYMENT-INFO.md`, inside the `warden-planning` checkout) — a
plain read-only `git clone`, not the WSL working copies these were actually built in.
Useful for anything that wants direct filesystem access on the Windows side (an IDE,
Antigravity, etc.) without needing WSL. These will drift from `main` the moment new
work lands anywhere — `git pull` each one before trusting it's current, the same as
you'd treat any other clone.

All four repos now have: CI running the real test suite on every PR (verified against a
real PR, not just YAML validity), branch protection on `main` (PR + 1 approval + passing
CI required; admin-bypassable by design so solo maintenance isn't blocked), LICENSE
(Apache 2.0), CONTRIBUTING.md, SECURITY.md (explicit "unaudited" statement + contact),
an approved-repo-pattern README, GitHub topics, and a set of planned next-step issues
created via a committed `gh` script (`.github/scripts/create-issues.sh` in each repo).

---

## Testnet deployment (`warden-contract`)

**Current — Phases 14+15+16, batch-deployed together** (dual velocity windows, trust
decay, flagged-address registry, guardian recovery — see the phase sections below):

```bash
WARDEN_NETWORK=testnet
WARDEN_RPC_URL=https://soroban-testnet.stellar.org
WARDEN_NETWORK_PASSPHRASE="Test SDF Network ; September 2015"
WARDEN_CONTRACT_ID=CD5QU2E6LOKFAZFESIZSAA4IENH5SZHJVU4Y6532WNZSXPZDYRKEEVUW
WARDEN_REFERENCE_ASSET=CDLZFC3SYJYDZT7K67VZ75HPJVIEUVNIXF47ZG2FB2RMQQVU2HHGCYSC
WARDEN_DEPLOY_LEDGER=4637276
WARDEN_ADMIN_ADDRESS=GCZLMMKEOPOG5OB5QRLGH5ZKG7ACQNKX7KTT6UTXPFHUPS7FFSFFU5YM
```

- **Contract explorer:** https://stellar.expert/explorer/testnet/contract/CD5QU2E6LOKFAZFESIZSAA4IENH5SZHJVU4Y6532WNZSXPZDYRKEEVUW
- **Wasm hash:** `6a5f341679e33bb7e37f6cd32312f6cb0a05be97bcc3925f7671dad8784c78d6`
- **Reasoning for reference asset choice:** native XLM's SAC, not a custom test-USDC —
  `warden-contract` never reads this address after `initialize`, so it isn't
  load-bearing for v1. Full reasoning in `07-warden-phase8-deployment.md` Step 3.
- **Why a redeploy for an additive change:** Phase 15 and 16 both added independent
  storage keys without touching `Policy`'s shape at all — unlike Phase 14, this wasn't
  a storage-compatibility break. It happened anyway because this contract has no
  admin-upgrade function by design, so any new function can only ever ship as a new
  contract instance.
- **Verified live, not just simulated locally**, immediately after deploying: `get_policy`
  null before / configured after `set_policy`+`add_trusted_recipient`, `evaluate()`
  `Allow` for a normal transfer, `RequireStepUp(FlaggedRecipient)` right after
  `add_flagged_address` on that same recipient and back to `Allow` after
  `remove_flagged_address`, `get_account_state` returning `Normal` and `get_guardians`
  correctly erroring `GuardiansNotConfigured` for a never-configured wallet.
- **`warden-app` and `warden-monitor` are both updated to this contract ID and to
  `warden-sdk` v0.3.0.** `warden-docs` is **not yet updated** — see Open items below.

**Retired — do not use.** Two earlier deployments, both superseded by the above:

| | |
|---|---|
| Phase 14 only | `CD25U7GYDNB7XUBEEN3OKZK2LY62ANSUJJPQ6SF2Y6DHQ5SQ3F7LSVUF` — no flagged-address registry or guardian/recovery functions |
| Pre-Phase-14 | `CBFQ752LFNC57U4KWDAEKNU43PLBWJ7M2B4ZRYUMCWL62JHJNUYJVMB5` — old `Policy` shape entirely |

Same pattern each time: this contract has no in-place migration path (no
admin-upgrade function, by design), so every redeploy means a new contract ID and the
one real Testnet wallet re-running `set_policy`/`add_trusted_recipient` from scratch.
Acceptable for Testnet with no real funds at stake; would not be acceptable for a
Mainnet deployment with real user data.

---

## Credentials & where they live

| What | Where | Notes |
|---|---|---|
| `deployer` identity (secret key) | `~/.config/stellar/identity/deployer.toml` inside WSL | Testnet-only, Friendbot-funded, no real value — but is the on-chain `admin` for the deployed contract above. Needed again if `warden-app`/`warden-sdk` reuse this identity, or to call any future admin-only action (none exist in v1). |
| GitHub auth (`gh`) | WSL: logged in as `Femology`, scopes `repo, workflow, gist, read:org` | Windows-side `gh` (outside WSL) is **not** authenticated — all repo creation happens from WSL. |
| WSL sudo password | Set by user, not stored here | Needed for any future `apt install` (e.g. new build deps for `warden-app`/`warden-monitor`). |

---

## Local toolchain (WSL — Ubuntu 24.04)

| Tool | Version | Notes |
|---|---|---|
| Rust | 1.98.1, edition 2021 | via `rustup`, `wasm32v1-none` target installed |
| `soroban-sdk` | exact `26.1.0` (pinned) | Cargo.toml uses `=26.1.0`, not caret range |
| Stellar CLI | 27.0.0 | Built from source (`cargo install stellar-cli --version 27.0.0`); needed `build-essential`, `pkg-config`, `libssl-dev`, `libdbus-1-dev`, `libudev-dev` as apt deps first |
| Node.js | 24.21.0 (LTS "Krypton"), native WSL install via NodeSource | Was leaking in from Windows via WSL interop (`/mnt/c/Program Files/nodejs`) before — installed properly native |
| `@stellar/stellar-sdk` | exact `17.0.1` | verified against npm registry directly |
| vitest | exact `5.0.0` | |
| TypeScript | exact `7.0.2` | |

Local repo path: `~/projects/warden-contract` (WSL native filesystem, not `/mnt/c/...`
— chosen for cargo build speed).

---

## Decisions made along the way (with reasoning, so they don't get silently re-litigated)

1. **soroban-sdk pinned to exact `26.1.0`**, not a caret range — the master PRD says
   "pin to 26.1.0" specifically.
2. **`#[contractevent]` used instead of the deprecated `env.events().publish`** — same
   on-chain topic/data wire shape, verified against the macro's actual source, with
   explicit `topics = [...]` overrides so struct-name auto-derivation (which appends
   the literal struct identifier, "Event" suffix included) doesn't drift from the
   spec'd event names.
3. **Default git branch is `main`**, not `master` — fixed on `warden-contract`, and
   `git config --global init.defaultBranch main` is now set so future repos default
   correctly without a fixup commit.
4. **`test_snapshots/` gitignored** — soroban-sdk's auto-generated test output,
   confirmed against the official `soroban-examples` repo's own `.gitignore`.
5. **Reference asset is native XLM's SAC**, not a custom test token — see deployment
   section above.

---

## `warden-sdk` — key decisions

1. **Built on `@stellar/stellar-sdk`'s high-level `contract.Client`/`AssembledTransaction`
   API** (verified empirically against the real deployed testnet contract — not
   guessed), rather than hand-rolling XDR/TransactionBuilder logic. This SDK fetches
   the contract's own on-chain spec once (cached) and uses it to encode args / decode
   results, so argument shapes can never silently drift from what's actually deployed.
2. **Error-code mapping bypasses `@stellar/stellar-sdk`'s built-in `errorTypes`
   machinery.** It reads Rust doc comments to build error messages, and
   `warden-contract`'s `WardenError` enum has none — so the SDK's own mapping comes
   back empty. Instead, `warden-sdk` regex-matches the raw host error text
   (`Error(Contract, #N)`, the exact format the Stellar CLI itself prints) and maps
   the code directly. Verified against the live contract, not assumed.
3. **Unit tests mock `@stellar/stellar-sdk/contract` at the module boundary**
   (`vi.mock`), not a fake RPC server — this tests `WardenClient`'s own logic
   (argument encoding, error mapping, XDR round trip) without needing to replicate
   real XDR wire format by hand.

## `warden-sdk` v0.1.3 — bugfix, found by actually running the docs' examples

`fromSignedXdr` reconstructed a transaction from XDR but never marked it as signed
(`AssembledTransaction.fromXdr` only sets `.built`, never `.signed`), so **every
`submit*` call threw `"The transaction has not yet been signed"` against a real
network**, regardless of correct signing. All 34 mocked unit tests passed throughout —
they mock the module boundary entirely and never exercise the real `.signed` check.
Fixed in [warden-sdk#5](https://github.com/Femology/warden-sdk/pull/5), tagged
`v0.1.3`. Superseded by `v0.2.x` below — this fix is carried forward, not undone.

## `warden-sdk` v0.2.0 → v0.2.1 — two more real bugs, same discipline

`v0.2.0` added the Phase 14 fields (hourly cap, trust decay, `Map`-shaped
`trustedRecipients`) but was only checked against mocked unit tests before being
pulled into `warden-app`/`warden-monitor`. Re-running `warden-docs`' examples against
the redeployed contract — the same "every example gets run for real" discipline that
caught the `v0.1.2` bug above — found two more, both fixed in `v0.2.1`
([warden-sdk#7](https://github.com/Femology/warden-sdk/pull/7)):

1. **`trustedRecipients` decoded with the wrong keys.** stellar-sdk decodes a Soroban
   `Map<Address, u64>` as an array of `[key, value]` tuples, not a plain object (an
   `Address` isn't a valid plain-object key). `v0.2.0`'s
   `{ ...raw.trusted_recipients }` on that array silently produced
   `{"0": [address, timestamp]}` — wrong shape *and* wrong content — instead of
   throwing. Fixed with `Object.fromEntries(...)`.
2. **`getPolicy` crashed instead of returning `null`** for a wallet with no policy
   (verified live, against a brand-new funded Testnet wallet), and `getVelocity` had no
   error handling at all. `.result` is a lazy getter — a reverted simulation throws
   when it's *accessed*, not inside `build()`'s own try/catch, so the raw error never
   reached `getPolicy`'s `instanceof WardenSdkError` check. Fixed with a shared
   `unwrapSimulated()` helper.

Both bugs passed every mocked unit test throughout, because the mocks modeled the
*wrong* shape for both (a plain-object fixture, and a `mockBuild` that rejects
synchronously instead of resolving with a throwing getter) — rewritten to match live
behavior. **Use `v0.2.1`, not `v0.2.0`, for anything that reads a policy.**
`warden-app` and `warden-monitor`'s dashboard are both bumped to it.

## `warden-contract` Phase 15 — flagged-address registry

`add_flagged_address`/`remove_flagged_address`, admin-gated: the caller must both sign
*and* be the exact address stored at `initialize` (`admin.require_auth()` alone only
proves identity, not privilege). `evaluate()` checks the recipient against this
registry before any policy-dependent check — a flagged recipient always gets
`RequireStepUp(FlaggedRecipient)`, regardless of amount, trust, or velocity headroom.
Starts empty in v1, no external feed wired up — stated plainly in the contract's own
README, and explicitly **not** an AI system anywhere in this stack.

## `warden-contract` Phase 16 — guardian and recovery subsystem

`set_guardians`, `propose_recovery`, `approve_recovery`, `execute_recovery`,
`cancel_recovery`. A wallet owner names up to 7 guardians who can, past a threshold and
a 48-hour timelock, move the account back toward normal operation without the owner's
own signature — `execute_recovery` has no `require_auth` call on any address at all,
which was verified two ways: a unit test that deliberately sabotaged the function with
an added auth check and confirmed the test failed before reverting, and a live call
submitted by a funded Testnet account with zero relationship to the target wallet,
which succeeded (rejected only for the correct business reason, `GuardiansNotConfigured`,
never for an auth reason). Guardians can never touch `Policy` or `trusted_recipients` —
verified: no guardian function calls the storage layer's `write_policy`.

**Three getters added after the fact:** `get_account_state`, `get_guardians`,
`get_recovery_proposal`. The original Phase 16 spec listed five state-changing
functions and zero reads, which would have left every downstream repo with no way to
read this data via a contract call at all — the same `get_policy`/`get_velocity`
pattern, applied here once the gap was noticed.

**Nothing escalates `AccountState` yet.** This phase only builds the
recovery-downward half of the state machine; automatic escalation is Phase 17's job
(an oracle layer, explicitly conditional on confirming a real third-party API vs.
building one from scratch — not started).

## `warden-contract` v0.3.0 — Phases 15+16, batch-deployed together

Not a storage-shape break like Phase 14 — Phase 15 and 16 both added independent
storage keys without touching `Policy` at all. Redeployed anyway because this contract
has no admin-upgrade function by design: any new function can only ever ship as a new
contract instance. See the "Testnet deployment" section above for the live
verification performed immediately after deploying.

## `warden-sdk` v0.3.0 — Phase 15/16 support, plus a real bug affecting every write call

Adds `build*`/`submit*` pairs for every Phase 15/16 function and the three new
getters. Two things found by actually exercising the new methods against the live
redeployed contract, not by trusting the mocks
([warden-sdk#8](https://github.com/Femology/warden-sdk/pull/8)):

1. **`AccountState` is fieldless in the contract but still wire-encoded as a tagged
   union** (`{ tag: "Normal" }`), not a bare string — assumed otherwise at first (a
   CLI's own pretty-printing was misleading). Broke in both directions: encoding a
   plain string into `propose_recovery`'s `target_state` argument threw "no such enum
   entry: undefined," and `get_account_state`'s real raw result is `{ tag: "Normal" }`,
   not `"Normal"`. Fixed by wrapping/unwrapping at the client boundary; every public
   method still takes/returns a plain string.
2. **A real bug in the shared `build()` helper, affecting every `build*` method, old
   and new** — it never checked for a reverted simulation before handing back signable
   XDR. A doomed write call (found via `buildProposeRecovery` with an invalid target
   state) still produced XDR that signed and submitted fine, then failed at the
   network with a bare `tx_malformed` instead of the specific `WardenError` the
   simulation already knew about. This retroactively explains an unresolved
   `tx_malformed` seen earlier this session against `add_trusted_recipient` on an
   already-trusted recipient — dismissed at the time as a one-off rather than chased
   to a cause; it wasn't a one-off. Fixed by having `build()` itself check `.result`
   (reusing the existing `unwrapSimulated()` helper) before returning, so every
   `build*` method now fails fast with a clean `WardenSdkError`.

`warden-app` and `warden-monitor`'s dashboard are both bumped to `v0.3.0`.

## `warden-contract` — WARDEN-PROTOCOL.md and the governance process (Phase 19)

A language-neutral spec of the protocol's rules, independent of Rust/Soroban/this
repo's source — `WARDEN-PROTOCOL.md` at the contract repo's root, versioned separately
(currently **1.0.0**) from the repo's own release tags via `CHANGELOG.md`. Covers every
data type, the exact `evaluate()` decision order, the account state diagram and every
legal transition, the flagged-address registry, the guardian/recovery subsystem, the
full event/error tables as stable consumer-facing interfaces, and an explicit
statement that Phase 17's signed-attestation schema doesn't exist because Phase 17
isn't active (rather than leaving that gap to be inferred from silence).

Caught one real mistake before it shipped: the first draft of the state-transition
diagram had several arrows backwards, pointing from `Normal` toward more restrictive
states — re-derived from the actual `AccountState` ordering and the `propose_recovery`
check rather than trusted on the first pass, and fixed all 10 legal transitions before
committing.

Any future change to what triggers a state transition or a step-up reason now
requires opening an issue via `.github/ISSUE_TEMPLATE/protocol-rule-change.md`
(three required fields: what changes, why, what it affects), acknowledged by a
maintainer, before any code is written — `CONTRIBUTING.md` and the README both point
to this now.

## Phase 18 — "Explain this" (Evomap / DeepSeek V4.1 Flash)

An "Explain this" action next to any step-up event in both `warden-app`
(`StepUpConfirmModal`) and `warden-monitor` (`WalletDrilldown`'s evaluation list).
Calls an LLM with **only** the structured on-chain facts for that one event (a
step-up's reason code and amount; the shared module also supports a state-transition
shape for whenever that gets a UI surface) — no wallet address, no history, no ability
to call any Warden function or change anything. The model's output is validated
against a fixed schema (`summary`, `factors`, `next_steps`) before ever being shown;
any deviation, including the model referencing a reason code that wasn't in the input,
discards the output and falls back to pre-written copy instead.

- Shared, secret-free schema/validation/fallback logic lives in `warden-sdk`
  (`buildExplainPrompt`, `validateExplanationResponse`, `fallbackExplanation`) —
  provider-agnostic, safe to import anywhere including client-side code.
- The actual model call (`https://api.evomap.ai/v1/chat/completions`, model
  `evomap-deepseek-v4-flash`) lives entirely in each app's own server-only
  `/api/explain` route, which is the only place `EVOMAP_API_KEY` is ever read. Kept
  deliberately out of `warden-sdk` itself, since that package is also imported by
  browser code and a key must never end up reachable there by accident.
- **The API key shared in this session's chat is exposed by definition and was never
  written to any file, prompt, or commit.** Both apps' `.env.local` have
  `EVOMAP_API_KEY=` left blank with a comment pointing at this. The feature works
  correctly either way (always falls back to pre-written explanations when the key is
  absent) — it just doesn't call the real model until the key is rotated on Evomap's
  dashboard and the **new** key is pasted in directly, never through chat again.
- Tests (10 per app, identical coverage) call each `/api/explain` route handler
  directly with a mocked Evomap response, covering exactly what was required
  explicitly: a fabricated/malformed model response falls back correctly, and a
  response referencing an ungrounded reason code is discarded rather than rendered.
- See `HOSTING.md` (`warden-monitor`) and both apps' READMEs for exactly which
  Vercel-hosted piece needs its own copy of `EVOMAP_API_KEY` — setting it on one does
  not cover the other, each has its own separate route.

## `warden-docs` — documentation site

| | |
|---|---|
| Repo | https://github.com/Femology/warden-docs |
| Platform | **GitBook**, not Mintlify |

Built for Mintlify first, per the original instruction. Switched to GitBook after
Mintlify's CLI (which bundles Puppeteer for local preview) failed twice in this
environment — an `EACCES` permission error on global install, then an `ETIMEDOUT`
network failure pulling Puppeteer's Chromium as a local devDependency. GitBook needs no
local CLI or build step at all: connect the repo at
[app.gitbook.com](https://app.gitbook.com) (Git Sync) and it publishes directly from
`README.md` / `SUMMARY.md` / `.gitbook.yaml` on every push to `main` — not yet
connected, so there's no live URL yet (same honest gap as `warden-app`'s Vercel
deployment).

Every code example in `developer-guide.md` was actually run against the live deployed
contract while writing it — that's how the `warden-sdk` bug above was found.

## `warden-contract` Phase 14 — dual velocity windows + trust decay

From `11-warden-feature-roadmap-phases-14-19.md` (this repo). Six commits, 27/27 tests
passing on real CI (verified, not just local `cargo test`):

- **Hourly velocity window** (`hourly_velocity_cap`, resets every 3600s) alongside the
  existing daily one. `evaluate()` checks both; `StepUpReason::HourlyVelocityExceeded`
  is new. `hourly_velocity_cap` must be `<= daily_velocity_cap` (`InvalidPolicyParams`
  otherwise) — a design decision made and documented in the code, not explicitly
  spelled out in the task text, since the feature's own stated rationale ("catches
  rapid-fire spending a single daily cap misses") only makes sense with an
  independently-meaningful (typically smaller) hourly threshold.
- **Trust decay.** `trusted_recipients` changed from `Vec<Address>` to
  `Map<Address, u64>` (`last_paid_at`), justified in a code comment. A new
  `trust_decay_seconds` policy field controls how long trust survives without a
  payment. `set_policy` now takes 6 params, not 4.
- **This is a breaking storage schema change with no upgrade path.** The full
  reasoning is in `warden-contract`'s own README under "Phase 14" / "Migration note."

### Redeployed — this section is now historical

This used to warn that the live Testnet contract was still running pre-Phase-14 code
while `main` had moved on. **That's resolved**: Phase 14 was deployed as a new contract
instance (`CD25U7GYDNB7XUBEEN3OKZK2LY62ANSUJJPQ6SF2Y6DHQ5SQ3F7LSVUF` — see the "Testnet
deployment" section above), the one real test wallet's policy was re-set from scratch
against it, and every downstream repo (`warden-sdk`, `warden-app`, `warden-monitor`,
`warden-docs`) is updated to match — including two real bugs in `warden-sdk` v0.2.0
found and fixed along the way (see the section above). The old contract ID
(`CBFQ752...`) is retired and its policy data is gone, exactly as this section
originally warned would happen.

## Open items / blockers

- **`EVOMAP_API_KEY` needs to be rotated before Phase 18 actually calls the real
  model anywhere.** The key shared in this session's chat is exposed by definition and
  was deliberately never written to any file — rotate it on Evomap's dashboard, then
  paste the new key directly into each app's own local `.env.local` (never into chat)
  and into each Vercel project's env vars. Until then, "Explain this" works correctly
  and safely, it just always shows the pre-written fallback copy, never the model's.
- **`warden-app`/`warden-monitor` still have no UI for Phase 15/16's data**, even
  though the SDK supports it end-to-end. `warden-monitor`'s `WalletDrilldown` could
  show flagged status, guardian config, and a pending recovery proposal, but doesn't.
  No page anywhere lists guardian-recovery or flagged-address *events* either — the
  indexer doesn't decode `guardians_set`/`recovery_*`/`address_flagged` yet, so there's
  nothing to list even if a page existed.
- **The Design System v2 UI redesign is mid-flight** — Step 1 (tokens, fonts, the
  persistent nav shell) shipped in `warden-app`; everything else in "Resume here"
  above is still open, including `warden-monitor`'s dashboard not having had any v2
  pass applied at all. The user is doing hands-on design work in Antigravity right
  now — check for in-progress, possibly-unpushed changes before touching UI code
  again.
- **`warden-app` and `warden-monitor` haven't had their GitHub Releases re-tagged**
  since Phase 14 — they're on `warden-sdk` v0.4.0 and the current contract ID in code
  and on `main`, but the `v0.1.0` release tag on each still reflects the original
  pre-Phase-14 state. Minor, cosmetic, not blocking anything functional.
- **`warden-app` has no public URL yet** (issue
  [#2](https://github.com/Femology/warden-app/issues/2)) — same for `warden-monitor`'s
  indexer and dashboard (issues
  [#2](https://github.com/Femology/warden-monitor/issues/2) and
  [#3](https://github.com/Femology/warden-monitor/issues/3)). `HOSTING.md` and both
  apps' READMEs are ready with the exact env vars each platform needs, `EVOMAP_API_KEY`
  included, whenever this actually happens.
- **`warden-docs` isn't connected to GitBook yet** — the repo is real, complete, and
  caught up through Phase 18; publishing it live is a one-time manual step at
  app.gitbook.com (needs your account, not something this session can do on your
  behalf).
