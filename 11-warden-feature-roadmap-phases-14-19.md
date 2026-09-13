# Warden — Feature Roadmap: Phases 14–19

**Sequencing constraint, non-negotiable:** none of this starts before the v1 lock ships — `warden-contract`'s deterministic policy (amount/velocity/allowlist/state) built, tested, deployed, and submitted to Wave. This file is what's next, not what's instead. Every phase below follows the same rule that was violated once already in this project: **spec before prompt, always** — where a phase needed a spec that didn't exist (Phase 16), it's written here, not deferred to whichever agent gets the prompt.

Apply the git workflow, coding discipline, and verify-before-using-any-API rules from `00-WARDEN-MASTER-PRD.md` section 4 to every phase below without restating them each time.

---

## Phase 14 — Multi-window velocity and trust decay

**Why this one first:** it's the actual answer to "can't we build a real risk engine" that doesn't touch the Stellar-load-bearing argument. Fully deterministic, no new trust surface, extends the existing contract rather than adding a component.

**What's added to `warden-contract`:**
- A second, shorter velocity window (e.g. hourly) alongside the existing 24h one — catches rapid-fire testing of a compromised wallet that a single daily cap misses on its own.
- Trust decay: a trusted recipient that hasn't been paid in a configurable number of days (e.g. 90) quietly stops counting as trusted — re-adding it, or a below-threshold amount, still works, it just no longer skips step-up by default.

```
Read 02-warden-phase5-contract-architecture.md and 03-warden-phase6-contract-agent-prompt.md
first — this phase extends that contract, it doesn't replace it.

Add: an hourly VelocityWindow alongside the existing daily one, evaluated with the
same logic (reset-on-expiry, accumulates regardless of decision) but on a 3600-
second window. evaluate() checks both windows and returns RequireStepUp if either
is exceeded.

Add a last_paid_at timestamp per trusted recipient (this changes trusted_recipients
from a Vec<Address> to a Vec<(Address, u64)> or an equivalent map — pick one and
justify it in a comment). On evaluate(), a recipient whose last_paid_at is older
than a configurable trust_decay_seconds no longer counts as trusted for the
new_recipient_requires_stepup check, even though it's still in the list. Update it
on every successful transfer to that recipient.

This changes existing stored data shape — write a migration note in the README
explaining that a wallet with policies set under the old schema needs to re-set
trust decay defaults, since this is Testnet and there's no real migration
tooling yet.

One commit per logical unit, same rules as before. Full test suite must pass,
including new tests for: hourly window triggering step-up while daily window still
has headroom, and a recipient decaying out of trust after the configured period.
```

## Phase 15 — On-chain flagged-address registry

**Why:** a real, deterministic way to make the policy meaningfully smarter without an oracle or ML — an admin-curated blocklist the contract checks directly.

```
Add a FlaggedAddress(Address) -> bool entry to warden-contract's storage. Add
add_flagged_address(admin, address) and remove_flagged_address(admin, address),
both admin.require_auth(), admin being the same address set at initialize.

evaluate() checks the recipient against this registry before anything else — a
flagged recipient always returns RequireStepUp(FlaggedRecipient), regardless of
amount, trust, or velocity. Add FlaggedRecipient as a new StepUpReason variant.

Be explicit in the README about what populates this registry in v1: it's manually
curated by the admin from a real external source you choose and periodically
sync (state which source, honestly, or state that it starts empty and this is
infrastructure for a future real feed) — this is not an AI system and must not be
described as one anywhere in the docs.

Test the admin-only gate explicitly: a non-admin address attempting to flag or
unflag an address must fail.
```

## Phase 16 — Guardian and recovery subsystem

**This is the piece that was missing a spec entirely — the frontend prompt assumed it existed, the contract prompt never defined it. Here's the spec, written now, before any prompt.**

### Design decisions, stated explicitly so nothing is ambiguous

- **Guardians can only be configured while the account is NORMAL or WATCH** — not while RESTRICTED, CHALLENGED, or FROZEN. This stops an attacker who has just compromised a wallet from immediately adding their own colluding guardian before the owner notices.
- **Recovery execution does NOT require the wallet owner's own signature.** This is the entire point of guardians — routing around a compromised owner key. Guardian threshold + elapsed timelock is sufficient to execute.
- **The wallet owner can cancel a pending recovery proposal at any time before it executes**, using their own signature. This protects against a guardian-majority collusion attack in the ordinary case where the owner's key is fine and a recovery was proposed maliciously or in error.
- **Guardians can only ever move the account toward less restriction, and only through this proposal/timelock/threshold path — never instantly, never by simple majority vote alone.**

### Storage
```rust
pub struct GuardianConfig {
    pub guardians: Vec<Address>,   // max 7
    pub threshold: u32,            // 1 <= threshold <= guardians.len()
}

pub struct RecoveryProposal {
    pub proposer: Address,
    pub target_state: AccountState,
    pub approvals: Vec<Address>,
    pub proposed_at: u64,
    pub timelock_seconds: u64,     // e.g. 172800 (48h), configurable per wallet
}
```

### Functions
- `set_guardians(wallet, guardians: Vec<Address>, threshold: u32)` — `wallet.require_auth()`. Errors if current state is not NORMAL or WATCH, if `guardians.len() > 7`, or if `threshold` is 0 or greater than `guardians.len()`.
- `propose_recovery(wallet, proposer: Address, target_state: AccountState)` — `proposer.require_auth()`. Errors if `proposer` is not in the wallet's guardian list, or if `target_state` isn't strictly less restrictive than the current state. Creates the proposal, starts the timelock, counts the proposer's own approval.
- `approve_recovery(wallet, guardian: Address)` — `guardian.require_auth()`. Errors if `guardian` isn't in the list or already approved this proposal.
- `execute_recovery(wallet)` — **no `require_auth` on the wallet itself.** Callable by anyone once `approvals.len() >= threshold` AND `now >= proposed_at + timelock_seconds`. Transitions the account to `target_state`, clears the proposal.
- `cancel_recovery(wallet)` — `wallet.require_auth()`. Clears any pending proposal. This is the owner's veto.

### Events
`guardians_set`, `recovery_proposed`, `recovery_approved`, `recovery_executed`, `recovery_cancelled` — all needed by `warden-monitor` to show a recovery in progress.

```
Build this exactly as specified above — do not add a mechanism for guardians to
change the account's policy or trusted recipients, only its state. Guardians
recover access to normal operation; they never gain the ability to configure how
the wallet behaves once recovered.

Write tests for: guardian configuration rejected while RESTRICTED or worse;
recovery executing without any signature from the wallet's own key, purely on
guardian threshold and elapsed time; the owner successfully cancelling a pending
proposal with their own signature; a non-guardian's approval attempt failing;
execution attempted before the timelock elapses failing.

Only after this contract-level spec is fully built and tested does the
Guardians/Recovery Center UI from the frontend prompt get built — it was written
before this spec existed and must now be checked against it, not assumed correct.
```

## Phase 17 — Oracle layer (CONDITIONAL — do not start without confirming this first)

```
Before writing any code for this phase: confirm explicitly whether Warden is
using a real third-party risk-scoring API (paid or free, with a real provider
name) versus building its own signal-collection and scoring service from scratch.
These are very different amounts of work and different security surfaces — do
not proceed on an assumption either way.

If a real third-party API is confirmed: warden-oracle becomes a thin adapter —
call the provider's API, map its response into Warden's reason-code format, sign
the result. Most of the "build your own signal engine" work in the earlier AI/
risk-intelligence document does not apply and should not be built.

If no such decision has been made yet: stop and get that decision before writing
anything in this phase. Do not default to building an in-house scoring service
just because a document described one.
```

If and when this proceeds, the non-negotiable invariant from the research still applies: the oracle may escalate state automatically; it may never de-escalate. De-escalation only ever happens through Phase 16's guardian/timelock path or the wallet owner's own action.

## Phase 18 — Contextual, grounded explanation feature

**Why this one's worth keeping from the AI-layer document:** it's genuinely good, and it doesn't touch authorization at all — pure UX, strictly downstream of decisions already made on-chain.

```
Add a single feature: an "Explain this" action next to any state transition or
step-up event in warden-app and warden-monitor. It calls an LLM with ONLY the
structured facts already on-chain for that event (previous state, new state,
which policy rule or reason code triggered it) and nothing else — no access to
call any Warden function, no ability to change state, no ability to invent a
reason code that isn't in the actual event data.

Require the LLM's output to match a fixed schema (summary, factors, next steps)
and validate it against that schema before rendering. If it produces anything
outside the schema or references a reason code that wasn't in the input, discard
the output and fall back to a plain, pre-written explanation per reason code
instead of showing anything ungrounded.

Test explicitly: feed it a fabricated/malformed model response and confirm the
UI falls back correctly rather than rendering it.
```

## Phase 19 — Public protocol spec and governance

**Why this matters for "complete open source project," specifically:** code alone isn't a protocol. Anyone building a compatible oracle, wallet integration, or alternative frontend needs a spec to build against that isn't "read the Rust source."

```
Write a WARDEN-PROTOCOL.md at the root of warden-contract, independent of any
one language's implementation: the account state diagram and every legal
transition, the exact signed-attestation schema (if Phase 17 is active), the
event schema every consumer can rely on staying stable, and a version number for
the protocol itself, separate from any single repo's release tag.

Add a CHANGELOG-driven process for future rule changes: any change to what
triggers a state transition or a step-up reason gets proposed as an issue using
a fixed template (what changes, why, what it affects) before it's implemented —
this is what makes the project reviewable by someone who didn't write it.
```

---

## What NOT to do with this file

Don't hand all six phases to an agent in one sitting. Each one still gets its own session, its own read-the-spec-first step, its own test gate — same discipline as every phase before this one. And don't start Phase 17 on autopilot; that's the one place in this whole roadmap where a wrong assumption (building an in-house ML pipeline nobody asked for) would waste real time.
