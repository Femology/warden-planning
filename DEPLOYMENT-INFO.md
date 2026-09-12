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

Where things actually stand on the UI complaint, plainly:

**Done and verified (real tests, real builds, real running server — not just written):**
- Freighter wallet connect works (`@creit.tech/stellar-wallets-kit`), passkey kept as a
  secondary "beta" option so the demo doesn't block on it.
- The font bug is fixed (was a WSL DNS problem, not a code bug) — Bricolage Grotesque
  and Instrument Sans actually load now.
- The chosen logo (colorful interlocking mark) exists in every required size/format and
  is wired into the landing page header.
- Six landing-page illustrations exist and three are wired in (`section-amount`,
  `section-recipient`, `section-velocity`, plus the noise texture as a page overlay).
- The wallet-connect modal is themed to match Warden's palette instead of the kit's
  generic light-mode default.

**Not done yet — this is the real next-session list, in order:**
1. `/policy`, `/transfer`, `/velocity` still use the old plain layout — no shared app
   shell (header/nav), none of the illustrations or logo wired in there yet.
2. No real multi-page navigation between the app screens once connected.
3. GSAP + Lenis motion not added yet — no page transitions, no scroll choreography.
4. The Paper Shaders hero effect not added — the threshold band is still a plain CSS
   drag slider, not the shader-driven version from the master brief.
5. `hero-threshold-band-idle.svg`, `og-share-image.png`, and the `StepUpConfirmModal`
   illustration (`stepup-moment.svg` etc. from `ASSET-BRIEF.md` section D) exist as
   files but aren't wired in anywhere yet.
6. `warden-monitor`'s dashboard hasn't had any of this pass applied.

**To pick this back up:** open a Claude Code session in `warden-app` (or point it at
this file first), say "continue the UI work from `DEPLOYMENT-INFO.md`," and go through
the list above in order — each one is independently useful, so stopping partway through
still leaves things better than before, same as this session did.

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
| `warden-contract` | https://github.com/Femology/warden-contract | Deployed to Testnet, Phases 14+15+16 (dual velocity/trust decay, flagged-address registry, guardian recovery). CI + branch protection. 58/58 tests. [v0.3.0](https://github.com/Femology/warden-contract/releases/tag/v0.3.0) — v0.2.0/v0.1.0 are retired earlier deployments |
| `warden-sdk` | https://github.com/Femology/warden-sdk | 52/52 tests. CI + branch protection. **Use tag `v0.3.0`** — adds Phase 15/16 support plus a real fail-fast bugfix (see below); earlier tags are all pre-Phase-15/16 or have known bugs |
| `warden-app` | https://github.com/Femology/warden-app | 14/14 tests. CI + branch protection. On `warden-sdk` v0.3.0, contract redeployed. [v0.1.0](https://github.com/Femology/warden-app/releases/tag/v0.1.0) (release not yet re-tagged). No public URL yet (issue #2) |
| `warden-monitor` | https://github.com/Femology/warden-monitor | 20/20 tests (indexer+dashboard). CI + branch protection. On `warden-sdk` v0.3.0, contract redeployed. [v0.1.0](https://github.com/Femology/warden-monitor/releases/tag/v0.1.0) (release not yet re-tagged). Not deployed yet (issues #2, #3) |
| `warden-docs` | https://github.com/Femology/warden-docs | Complete GitBook site (6 pages), updated for Phase 14. **Not yet updated for Phase 15/16** — flagged below as an open item. Not yet connected to app.gitbook.com — no live URL yet |
| `warden-planning` | https://github.com/Femology/warden-planning | This file, the master PRD, and every phase build prompt. Clone this first on a new machine. |

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

- **`warden-docs` has not been updated for Phase 15/16 yet.** It still documents the
  Phase-14-only contract ID and shape — the flagged-address registry and guardian
  recovery subsystem aren't mentioned anywhere in it. Same "every example gets run for
  real" discipline should apply when this happens.
- **`warden-app`/`warden-monitor` haven't wired up any UI for Phase 15/16 either.**
  The SDK supports flagging addresses and guardian recovery end-to-end; nothing in
  either app's UI calls any of it yet. `warden-monitor`'s `WalletDrilldown` in
  particular could show flagged status, guardian config, and a pending recovery
  proposal, but doesn't.
- **`warden-app` and `warden-monitor` haven't had their GitHub Releases re-tagged**
  since Phase 14 — they're on `warden-sdk` v0.3.0 and the current contract ID in code
  and on `main`, but the `v0.1.0` release tag on each still reflects the original
  pre-Phase-14 state. Minor, cosmetic, not blocking anything functional.
- **`warden-app` has no public URL yet** (issue
  [#2](https://github.com/Femology/warden-app/issues/2)) — same for `warden-monitor`'s
  indexer and dashboard (issues
  [#2](https://github.com/Femology/warden-monitor/issues/2) and
  [#3](https://github.com/Femology/warden-monitor/issues/3)).
- **`warden-docs` isn't connected to GitBook yet** — the repo is real and complete;
  publishing it live is a one-time manual step at app.gitbook.com (needs your account,
  not something this session can do on your behalf).
