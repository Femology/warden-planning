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
| `warden-contract` | https://github.com/Femology/warden-contract | Deployed to Testnet. CI + branch protection. [v0.1.0](https://github.com/Femology/warden-contract/releases/tag/v0.1.0) |
| `warden-sdk` | https://github.com/Femology/warden-sdk | 34/34 tests. CI + branch protection. **Use tag `v0.1.3`** — v0.1.2 has a real submit* bug, see below; v0.1.0/v0.1.1 are stale build tags |
| `warden-app` | https://github.com/Femology/warden-app | 14/14 tests. CI + branch protection. [v0.1.0](https://github.com/Femology/warden-app/releases/tag/v0.1.0). No public URL yet (issue #2) |
| `warden-monitor` | https://github.com/Femology/warden-monitor | 20/20 tests (indexer+dashboard). CI + branch protection. [v0.1.0](https://github.com/Femology/warden-monitor/releases/tag/v0.1.0). Not deployed yet (issues #2, #3) |
| `warden-docs` | https://github.com/Femology/warden-docs | Complete GitBook site (6 pages). Every dev-guide example run against the live contract. Not yet connected to app.gitbook.com — no live URL yet |
| `warden-planning` | https://github.com/Femology/warden-planning | This file, the master PRD, and every phase build prompt. Clone this first on a new machine. |

All four repos now have: CI running the real test suite on every PR (verified against a
real PR, not just YAML validity), branch protection on `main` (PR + 1 approval + passing
CI required; admin-bypassable by design so solo maintenance isn't blocked), LICENSE
(Apache 2.0), CONTRIBUTING.md, SECURITY.md (explicit "unaudited" statement + contact),
an approved-repo-pattern README, GitHub topics, and a set of planned next-step issues
created via a committed `gh` script (`.github/scripts/create-issues.sh` in each repo).

---

## Testnet deployment (`warden-contract`)

```bash
WARDEN_NETWORK=testnet
WARDEN_RPC_URL=https://soroban-testnet.stellar.org
WARDEN_NETWORK_PASSPHRASE="Test SDF Network ; September 2015"
WARDEN_CONTRACT_ID=CBFQ752LFNC57U4KWDAEKNU43PLBWJ7M2B4ZRYUMCWL62JHJNUYJVMB5
WARDEN_REFERENCE_ASSET=CDLZFC3SYJYDZT7K67VZ75HPJVIEUVNIXF47ZG2FB2RMQQVU2HHGCYSC
WARDEN_DEPLOY_LEDGER=4598184
WARDEN_DEPLOY_TX_HASH=a508498a29318d06c08d4dd5e8f8e5d480ae2b95f507affac1bc521dafc81f77
WARDEN_ADMIN_ADDRESS=GCZLMMKEOPOG5OB5QRLGH5ZKG7ACQNKX7KTT6UTXPFHUPS7FFSFFU5YM
```

- **Contract explorer:** https://stellar.expert/explorer/testnet/contract/CBFQ752LFNC57U4KWDAEKNU43PLBWJ7M2B4ZRYUMCWL62JHJNUYJVMB5
- **`initialize` tx:** https://stellar.expert/explorer/testnet/tx/21d984c8473b0a67f80c12444446b53ca4dedefecfd8e594d3698c71b08b9bf6
- **Deploy tx:** https://stellar.expert/explorer/testnet/tx/a508498a29318d06c08d4dd5e8f8e5d480ae2b95f507affac1bc521dafc81f77
- **Wasm hash:** `70a7f7ec983eef92c6ba624ad08b2ed4256260ccc7242eb21b72bacc19d24324`
- **Reasoning for reference asset choice:** native XLM's SAC, not a custom test-USDC —
  `warden-contract` never reads this address after `initialize`, so it isn't
  load-bearing for v1. Full reasoning in `07-warden-phase8-deployment.md` Step 3.

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
`v0.1.3`. `warden-app` and `warden-monitor`'s dashboard are both bumped to it.
**Use `v0.1.3`, not `v0.1.2`, for anything that actually submits a transaction.**

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

### This directly affects the live Testnet deployment

The contract ID in this file's "Testnet deployment" section above
(`CBFQ752LFNC57U4KWDAEKNU43PLBWJ7M2B4ZRYUMCWL62JHJNUYJVMB5`) is running the
**pre-Phase-14 code** — it has real policy data under the old schema from earlier
testing, and that data cannot be read by the new contract code (different struct
shape). **This Phase 14 code has not been deployed anywhere yet.** Deploying it means
a new contract ID, not an upgrade of the existing one, and re-running Phase 8's deploy
steps (see `07-warden-phase8-deployment.md`) — including re-setting every wallet's
policy from scratch under the new deployment. Until that redeploy happens, the live
Testnet contract and the `main` branch of `warden-contract` are running different
schemas — worth knowing before pointing `warden-sdk`/`warden-app` at either one.

## Open items / blockers

- **`warden-contract`'s Phase 14 code needs a fresh Testnet deployment** (new contract
  ID — see above) before anything downstream can use the hourly window or trust decay.
- **`warden-app` has no public URL yet** (issue
  [#2](https://github.com/Femology/warden-app/issues/2)) — same for `warden-monitor`'s
  indexer and dashboard (issues
  [#2](https://github.com/Femology/warden-monitor/issues/2) and
  [#3](https://github.com/Femology/warden-monitor/issues/3)).
- **`warden-docs` isn't connected to GitBook yet** — the repo is real and complete;
  publishing it live is a one-time manual step at app.gitbook.com (needs your account,
  not something this session can do on your behalf).
