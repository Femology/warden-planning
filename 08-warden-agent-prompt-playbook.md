# Warden — Agent Prompt Playbook, Phase by Phase

Every prompt below is copy-paste ready for Claude Code. Run phases in order. **Never start a phase in the same session that finished the previous one — start fresh, let the new session read the files, and let it prove the previous phase's tests still pass before building on it.**

Phases 1–3 (ecosystem research, idea generation, critical review) are already done — that's how Warden got picked. This file starts at Phase 4, which is already built (files `01`–`02`), and carries all the way to Phase 13.

## The rule that applies to every single phase in this file

> **A phase is not done until its tests pass, in full, with the output pasted back to you.** Not "should pass." Not "I've implemented it." Run the suite, show the result. If anything fails, fix it before saying the phase is complete — don't report partial success as success.

Every prompt below ends with this same gate restated, on purpose. Don't skip pasting it because it looks repetitive — it's the thing that stops an agent from quietly declaring victory on a red test suite.

---

## Phase 0 — Orientation

**Prerequisite:** none. Run this in a brand new folder before any repo exists.

```
Read 00-WARDEN-MASTER-PRD.md in full, then 01-warden-phase4-scope-and-brand.md and
02-warden-phase5-contract-architecture.md. Do not write any code.

Summarize back to me: what Warden is, why Stellar is load-bearing (not decorative)
here, the four-repo build order and dependency direction, the three honest
limitations in section 3, and the non-negotiable rules in section 4. Then stop and
wait for me to confirm before doing anything else.
```

**Definition of done:** their summary correctly states that step-up is app-enforced not cryptographically enforced in v1, that the monitor never influences decisions, and that no monetary value may ever touch a float. If any of those three are missing or wrong, correct it before moving on — it means the file wasn't actually read carefully.

---

## Phase 6 — Build `warden-contract`

**Prerequisite:** Phase 0 confirmed.

```
Read 03-warden-phase6-contract-agent-prompt.md — this is your complete system prompt
for this repo. Before writing anything, verify the current Rust edition, MSRV, and
wasm target triple against the actual soroban-sdk 26.1.0 crate's own manifest/docs —
do not assume wasm32-unknown-unknown vs wasm32v1-none, check which one 26.1.0 uses.

Create a GitHub repo called warden-contract with gh CLI, initialize it here, and
follow the 26-commit build sequence exactly, one commit at a time, pushing
immediately after each. Never git add . — stage files by name.

After every 5 commits, stop, run cargo test, paste the full output, and wait for me
before continuing.

A phase is not done until its tests pass, in full, with the output pasted back to
me. Do not report the contract complete unless every test from commits 7, 9, 11,
13, 15–21, and 24 is green in one final full run of cargo test.
```

**Definition of done:** `cargo test` shows all tests passing, 26 commits pushed, no `unwrap()` outside `#[cfg(test)]`, no `panic!()` for an expected failure path.

---

## Phase 8 — Deploy `warden-contract`

**Prerequisite:** Phase 6 fully green.

```
Read 07-warden-phase8-deployment.md and follow it step by step. Diagnose any error
from its actual text before trying a fix — don't reinstall the toolchain for a PATH
problem. Stop after Step 4 (deploy) and show me the contract ID before calling
initialize — that step is permanent.

Once initialize and the verify step succeed, give me the full env var block from
the file, filled in with real values, and confirm you've captured
WARDEN_DEPLOY_LEDGER specifically — it can't be recovered later if lost.
```

**Definition of done:** you have the filled-in env block in hand, `get_policy` on a fresh address returns a clean `PolicyNotFound`, and the deploy ledger number is written down somewhere outside the terminal history.

---

## Phase 7a — Build `warden-sdk`

**Prerequisite:** Phase 8's env block in hand.

```
Read 00-WARDEN-MASTER-PRD.md and 04-warden-phase7-sdk-agent-prompt.md.
Here is the deployed contract's config: [paste the Phase 8 env block]

Verify the current published version of @stellar/stellar-sdk before pinning it —
confirm it's still the consolidated package for Soroban RPC calls. Do the same for
vitest.

Create the warden-sdk GitHub repo and follow the 20-commit build sequence exactly,
one commit at a time, pushing after each.

A phase is not done until its tests pass, in full, with the output pasted back to
me. Run the full vitest suite at the end and paste the result — every test from
commits 5, 9, 11, 13, 15, and 18 must be green.
```

**Definition of done:** full `vitest run` green, no monetary value anywhere routed through `parseFloat`/`Number` (grep for it if in doubt), `submitEvaluate` returns a correctly typed `Decision`.

---

## Phase 7b — Build `warden-app`

**Prerequisite:** `warden-sdk` published or git-installable at a tagged commit.

```
Read 00-WARDEN-MASTER-PRD.md — section 6 (design system) governs every screen you
build, not just the landing page. Read it before writing any component, and
re-check against it before you consider any UI commit finished, not just once at
the start.

Then read 05-warden-phase7-app-agent-prompt.md, your complete system prompt for
this repo. Verify passkey-kit's current package name and API for wallet creation,
existence check, and signing before writing against it — do not invent method
names.

Create the warden-app GitHub repo and follow the 21-commit build sequence, one
commit at a time, pushing after each.

Before you tell me the UI is done, self-audit it against section 6.2's banned-
patterns list, item by item, and confirm none apply. Then check it against 6.9's
quality floor: keyboard focus visible everywhere, WCAG AA contrast against
--ink-900, every async action has a distinct loading and error state, modals are
focus-trapped and Escape-dismissible, prefers-reduced-motion is respected including
on the hero. Report the result of that checklist explicitly — don't just say
"looks good."

A phase is not done until its tests pass, in full, with the output pasted back to
me. Every test from commits 9, 12, 16, 17, and 19 must be green, and the three demo
scenarios (allowed, step-up by amount or new recipient, step-up by velocity) must
each be manually walkable start to finish without an error.
```

**Definition of done:** test suite green, banned-patterns self-audit reported clean, quality-floor checklist reported clean, all three demo scenarios walkable live against the deployed Testnet contract — not mocked.

---

## Phase 7c — Build `warden-monitor`

**Prerequisite:** `warden-contract` deployed (Phase 8) and emitting real events (i.e. `warden-app`'s demo has generated at least a few transactions to index).

```
Read 00-WARDEN-MASTER-PRD.md section 6 — this dashboard follows the same design
system as warden-app, not a generic admin-panel look. Then read
06-warden-phase7-monitor-agent-prompt.md, your complete system prompt for this repo.

Verify Soroban RPC's current getEvents pagination parameters and
@stellar/stellar-sdk's current event-decoding helpers before writing the indexer —
do not assume a shape.

Create the warden-monitor GitHub repo and follow the 19-commit build sequence, one
commit at a time, pushing after each.

Before declaring this done, confirm explicitly: no endpoint or code path anywhere
in this repo writes to warden-contract; current policy/velocity on the wallet
drill-down page comes from a live warden-sdk read, never from the indexer's cache;
and the retention-window guard actually warns rather than silently showing gaps as
complete data.

A phase is not done until its tests pass, in full, with the output pasted back to
me. Every test from commits 4, 7, 11, and 17 must be green.
```

**Definition of done:** test suite green, the three explicit confirmations above stated back to you in writing, dashboard showing real indexed events from the Testnet deploy.

---

## Phase 9 — Hosting & Service Topology

**Prerequisite:** all four repos built and passing locally.

**Manual steps only you can do** (the agent has no browser or account access): create a Vercel project for `warden-app`, a Vercel project for the `warden-monitor` dashboard, and a Render (or equivalent) service for the `warden-monitor` indexer. Connect each to its GitHub repo.

```
Write render.yaml for the warden-monitor indexer with a persistent disk mounted at
the path DB_PATH points to — SQLite on an ephemeral filesystem loses all indexed
history on every redeploy, so this is not optional. Verify Render's current
persistent-disk feature and syntax rather than assuming an old config shape.

Write a HOSTING.md in warden-monitor with an explicit topology diagram: user →
warden-app (Vercel) → Stellar RPC directly for reads/writes; dashboard (Vercel) →
indexer API (Render) → SQLite (persistent disk) ← polls ← Stellar RPC. List every
environment variable each platform needs, matching the names from Phase 8's env
block exactly.

Do not hardcode any contract ID, RPC URL, or secret into these config files —
reference environment variables only, and note explicitly that Next.js
NEXT_PUBLIC_ variables are inlined at build time, so changing one requires a new
Vercel build, not just a restart — this exact class of bug (still calling the old
value after changing an env var) is worth guarding against explicitly in the docs.
```

**Definition of done:** `warden-app` loads at a real Vercel URL and can read a live policy from the deployed contract. The dashboard loads at a real Vercel URL and shows real data from the Render-hosted indexer. Trigger a redeploy of the indexer and confirm its event history survived — that's the actual test of whether the persistent disk is configured correctly, not just whether it deployed.

---

## Phase 10 — Repo Hygiene for Program Approval

**Prerequisite:** Phase 9 live and working. This phase exists because two prior submissions were rejected — treat it as the phase that actually determines the outcome.

```
For each of the four repos, in order:

1. Add a .github/workflows/ci.yml running the repo's real test suite (cargo test
   for warden-contract; the relevant test command for the others) on every pull
   request. This is required before branch protection can reference real status
   check names — don't reference a CI job name that doesn't exist yet.

2. Configure branch protection on main: require pull requests, require the CI
   check from step 1 to pass, require at least one approval.

3. Add CONTRIBUTING.md and SECURITY.md. SECURITY.md must include a responsible-
   disclosure contact and an explicit "unaudited, use at your own risk" statement —
   this project has not had a third-party audit.

4. Rewrite the README to match the pattern of the strongest approved Stellar repos:
   a banner/logo image, badges (build status, license), a maintainer table with a
   real contact method, a link to wherever the project's community lives, a
   concise architecture explanation (not a wall of text), practical copy-pasteable
   quickstart commands, a contributing section, and a contributors-credits image
   (contrib.rocks or equivalent).

5. Add relevant GitHub topics for discoverability.

6. Write a gh CLI script that creates every planned next-step issue for this repo
   in one run. Each issue needs a commit-style title, complexity and type labels,
   and a body with Summary, Acceptance Criteria as checkboxes, and Tech Stack.
   Split issues by component so a contributor can find scoped work.

7. Once everything above is committed and pushed, tag a v0.1.0 release. The release
   body must include the deployed contract ID, network, and a link to the live app
   — real addresses, not placeholders.

A phase is not done until its tests pass, in full, with the output pasted back to
me — confirm the CI workflow actually runs and passes on a real PR before moving
to the next repo, don't just confirm the YAML is syntactically valid.
```

**Definition of done, per repo:** a real PR triggers CI and it passes; branch protection actually blocks a direct push to `main` (test this once, deliberately); README visually matches the approved-repo pattern, not a generic technical readme; `v0.1.0` tag exists with real deployed addresses in its body.

---

## Phase 11 — Documentation Site

**Prerequisite:** Phase 10 complete on all four repos.

```
Build a documentation site (Mintlify is a strong, current example of good developer
docs done well — verify its current setup process rather than assuming an old one;
GitBook is an acceptable alternative) covering both a non-technical reader and a
technical reviewer:

- Introduction: what Warden is, the real cited problem (use the BMONI complaint
  quote and the specific user-facing pain it addresses), how it works step by step
- Protocol mechanics: the full evaluate() decision lifecycle, the velocity window
  behavior including its known fixed-window limitation, worked numeric examples
  with real amounts — not placeholders
- Contract reference: all 7 functions, exact parameters, return types, what
  triggers each, who can call it
- End-user guide: setting a policy, managing trusted recipients, understanding a
  step-up prompt — written in plain language, no jargon
- Developer guide: installing warden-sdk, environment variables, real code
  examples that actually run against the deployed Testnet contract, not
  invented snippets
- Contributing guide, linking back to each repo's CONTRIBUTING.md

Writing style: no AI-sounding filler, no inflated language, real numbers over vague
claims, short direct sentences — match the tone of the master PRD itself.

A phase is not done until every code example on the site has actually been run
against the live Testnet deployment and confirmed to work exactly as written —
paste me the output of running at least three of them before declaring this done.
```

**Definition of done:** site is live at a real URL, every code sample has been verified to actually execute, and a reviewer with zero prior context could follow the developer guide start to finish without hitting a wrong instruction.

---

## Phase 12 — Submission

**Prerequisite:** Phases 9–11 complete.

```
First, search live to confirm warden-contract is not already showing as approved
in the Stellar Wave program — don't assume its status from memory.

Then assemble: the live warden-app URL, all four repo URLs, an on-chain block-
explorer link for the deployed contract (e.g. Stellar Expert), the docs site URL,
and note that a short demo video showing all three scenarios end to end still
needs to be recorded manually — that part isn't something you can do.

Write the repo-relationship description explaining how the four repos connect
technically (contract → sdk → app / monitor).

Write the "planned issues" description, grounded in the real issues already
created in Phase 10 — organized by repo, specific enough to signal ongoing work,
not a one-off submission.

Write the project description for the submission form: plain English, one
paragraph, states the real problem with its cited figure, the mechanism, the
technical foundation, and why Stellar specifically is load-bearing.
```

**Important context, not for the agent — for you:** since Wave 7, rejection is not final. There's an in-app appeal (Maintainers → Orgs and Repos → Appeal), available 2 weeks after rejection, with a 1-month cooldown between attempts and a max of 3 appeals per repo. If this submission is rejected, that's the next move — not a rewrite from scratch, and not an email or Discord message, which get ignored.

**Definition of done:** every link in the submission actually resolves and shows real, working content — click each one yourself before submitting, don't trust that they're correct because the agent wrote them.

---

## Phase 13 — Post-Approval Iteration

**Prerequisite:** approved, or appealing.

Use this prompt shape for every new gap found from here on:

```
A new gap has been found: [describe it]. Before writing an issue: is this a quick
addition, or does it touch core architecture — say which, honestly, before scoping
it. If it spans more than one repo, write coordinated issues with an explicit
"Depends on" cross-reference between them, and confirm the dependency order before
building anything — sequencing this wrong (e.g. shipping an app feature before its
contract dependency is deployed) creates bugs that look like regressions but
aren't.
```

**Definition of done, every time:** the issue states plainly whether it's a quick fix or a real architecture change, and nothing gets built until the dependency order across repos is confirmed.
