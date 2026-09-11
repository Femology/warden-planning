# Warden — Phase 5: Contract Architecture

## Contract enumeration
One contract for v1, per the Phase 4 scope decision: **`warden-contract`**. Single responsibility: store per-wallet policy rules, track per-wallet spending velocity, and decide whether a proposed transfer can proceed unassisted or needs step-up confirmation.

No second contract exists in v1. A step-up-signer contract was considered and deliberately deferred — see "Deferred, not forgotten" at the end of this file.

## Dependency graph and build order
Single contract, no cross-contract calls in v1. Build strictly in this order — later pieces depend on earlier ones:

1. Data types: `Policy`, `VelocityWindow`, `Decision`, `StepUpReason`, `DataKey`, `WardenError`
2. Storage helper functions (get/set policy, get/set velocity, TTL-extension helpers)
3. `initialize`
4. `set_policy`
5. `add_trusted_recipient` / `remove_trusted_recipient`
6. `evaluate` (depends on every storage helper above)
7. `get_policy` / `get_velocity` (read-only views)
8. Tests, added in the same order as the functions they cover

## Data model

### Storage keys
```rust
pub enum DataKey {
    Admin,              // instance storage
    ReferenceAsset,     // instance storage
    Policy(Address),    // persistent storage, keyed by wallet
    Velocity(Address),  // persistent storage, keyed by wallet
}
```
`Admin` and `ReferenceAsset` are contract-level config, set once at `initialize` and never changed in v1 — use instance storage with instance TTL extension. `Policy` and `Velocity` are per-wallet and must use persistent storage with TTL extension on every write, since a wallet may go months without a transaction and its policy must not expire from state.

### Structs
```rust
pub struct Policy {
    pub owner: Address,
    pub max_no_stepup: i128,               // per-tx ceiling before step-up is required
    pub daily_velocity_cap: i128,          // cumulative ceiling per rolling 24h window
    pub new_recipient_requires_stepup: bool,
    pub trusted_recipients: Vec<Address>,
    pub updated_at: u64,                   // ledger timestamp of last policy change
}

pub struct VelocityWindow {
    pub window_start: u64,      // ledger timestamp the current window began
    pub cumulative_amount: i128,
    pub tx_count: u32,
}
```

### Enums
```rust
pub enum Decision {
    Allow,
    RequireStepUp(StepUpReason),
}

pub enum StepUpReason {
    AmountExceeded,
    NewRecipient,
    VelocityExceeded,
}
```

### Errors
```rust
#[contracterror]
pub enum WardenError {
    NotInitialized = 1,
    AlreadyInitialized = 2,
    PolicyNotFound = 3,
    InvalidAmount = 4,
    InvalidPolicyParams = 5,
    RecipientAlreadyTrusted = 6,
    RecipientNotTrusted = 7,
}
```

## Public functions

### `initialize(env: Env, admin: Address, reference_asset: Address) -> Result<(), WardenError>`
- **Auth:** `admin.require_auth()`
- **Behavior:** fails `AlreadyInitialized` if `DataKey::Admin` already set. Stores `admin` and `reference_asset` in instance storage. Bumps instance TTL.
- **Real step it maps to:** one-time contract setup at deploy time.

### `set_policy(env: Env, wallet: Address, max_no_stepup: i128, daily_velocity_cap: i128, new_recipient_requires_stepup: bool) -> Result<(), WardenError>`
- **Auth:** `wallet.require_auth()`
- **Behavior:** validate `max_no_stepup >= 0` and `daily_velocity_cap >= max_no_stepup` (a velocity cap below the per-tx threshold is nonsensical — reject with `InvalidPolicyParams`, don't silently clamp it). If no policy exists yet, create one with an empty `trusted_recipients` list. If one exists, update these fields in place and leave `trusted_recipients` untouched (that list is only managed by the two functions below). Set `updated_at` to the current ledger timestamp. Persist with TTL extension. Emit `policy_set`.
- **Real step it maps to:** the wallet owner setting or changing their risk tolerance — the core settings action.

### `add_trusted_recipient(env: Env, wallet: Address, recipient: Address) -> Result<(), WardenError>`
- **Auth:** `wallet.require_auth()`
- **Behavior:** load policy (`PolicyNotFound` if `set_policy` was never called — a wallet must configure a policy before it can trust anyone). Error `RecipientAlreadyTrusted` if already present. Otherwise append, update `updated_at`, persist, extend TTL. Emit `recipient_trusted`.
- **Real step it maps to:** user saves a payee — the ordinary "add to trusted contacts" action.

### `remove_trusted_recipient(env: Env, wallet: Address, recipient: Address) -> Result<(), WardenError>`
- **Auth:** `wallet.require_auth()`
- **Behavior:** load policy, error `RecipientNotTrusted` if absent, otherwise remove and persist. Emit `recipient_untrusted`.
- **Real step it maps to:** user revokes a payee (compromised address, wrong entry, no longer needed).

### `evaluate(env: Env, wallet: Address, recipient: Address, amount: i128) -> Result<Decision, WardenError>`
This is the core function — the one called at the moment a transfer is attempted.
- **Auth:** `wallet.require_auth()`. The wallet's own existing signer (e.g. its passkey) must authorize this call as part of the same transaction the transfer belongs to — this is what lets `evaluate` be invoked from inside the smart wallet's own auth flow rather than as a separate, spoofable side-channel.
- **Behavior, in order:**
  1. Load policy for `wallet`. `PolicyNotFound` if none configured. **Deliberate design choice:** an unconfigured wallet gets an error, not a silent default decision — "no policy" is a distinct state the calling app must handle explicitly (e.g. by defaulting to always-require-step-up at the app layer), not something the contract should paper over.
  2. Validate `amount > 0` (`InvalidAmount` otherwise).
  3. Load `VelocityWindow` for `wallet`, or start a fresh one if none exists. If `env.ledger().timestamp() - window.window_start >= 86400`, reset it (`window_start = now`, `cumulative_amount = 0`, `tx_count = 0`). **Known v1 simplification:** this is a fixed, reset-on-expiry window, not a continuously sliding one. A wallet could in principle spend up to its cap right before a reset and again right after. This is acceptable for v1 and should be written up as a stated limitation, not hidden as an oversight.
  4. Determine the decision, checking in this order and returning on first match:
     - if `new_recipient_requires_stepup` is true and `recipient` is not in `trusted_recipients` → `RequireStepUp(NewRecipient)`
     - else if `amount > max_no_stepup` → `RequireStepUp(AmountExceeded)`
     - else if `cumulative_amount + amount > daily_velocity_cap` → `RequireStepUp(VelocityExceeded)`
     - else → `Allow`
     (v1 returns only the first matching reason. Surfacing all simultaneously-true reasons is a reasonable v2 improvement, not a v1 requirement — nothing in the current user flow needs more than one reason at a time.)
  5. **Regardless of the decision**, update the velocity window: `cumulative_amount += amount`, `tx_count += 1`, persist with TTL extension. This is deliberate — a transfer that triggered step-up and was then completed by the user still happened and must count toward velocity; only counting `Allow`-ed transfers would let someone reset their effective velocity cap just by making every transfer trigger step-up.
  6. Emit `evaluation_allowed(wallet, recipient, amount)` on `Allow`, or `stepup_required(wallet, recipient, amount, reason)` on `RequireStepUp`.
  7. Return the `Decision`.

### `get_policy(env: Env, wallet: Address) -> Result<Policy, WardenError>`
- **Auth:** none — public read. **Stated trade-off:** policy data is not secret (it's config, not a secret, and everything on a public ledger is inspectable anyway); v1 does not gate this. If a future version needs to hide policy details from other parties, that's a real requirement to design for explicitly, not something to bolt on later.
- `PolicyNotFound` if none set.
- **Real step it maps to:** the app's settings screen displaying current policy back to the owner.

### `get_velocity(env: Env, wallet: Address) -> Result<VelocityWindow, WardenError>`
- **Auth:** none — public read, same reasoning as above.
- **Real step it maps to:** `warden-app` and `warden-monitor` both need to show how close a wallet is running to its velocity cap.

## Events
Every event a real consumer (`warden-monitor`) needs to build a working dashboard — nothing decorative:

| Event | Topics | Data |
|---|---|---|
| `policy_set` | `("policy_set", wallet)` | `(max_no_stepup, daily_velocity_cap, new_recipient_requires_stepup)` |
| `recipient_trusted` | `("recipient_trusted", wallet)` | `recipient` |
| `recipient_untrusted` | `("recipient_untrusted", wallet)` | `recipient` |
| `evaluation_allowed` | `("eval_allowed", wallet)` | `(recipient, amount)` |
| `stepup_required` | `("stepup_req", wallet)` | `(recipient, amount, reason)` |

Use `env.events().publish((topic_symbols...), data_tuple)` at the point of each state change described above — not batched, not deferred.

## Interface surface `warden-sdk` must wrap
This is the exact 1:1 contract-call surface the TypeScript SDK exposes, so Phase 7 (the app system prompt) can be written against a fixed target:
- `setPolicy(wallet, maxNoStepUp, dailyVelocityCap, newRecipientRequiresStepUp)`
- `addTrustedRecipient(wallet, recipient)`
- `removeTrustedRecipient(wallet, recipient)`
- `evaluate(wallet, recipient, amount) -> Decision`
- `getPolicy(wallet) -> Policy`
- `getVelocity(wallet) -> VelocityWindow`

`initialize` is deploy-time only and is **not** part of the public SDK surface — it's a one-off script, not something an integrating app ever calls.

The SDK also owns translating between the portable policy-rule JSON shape (from Phase 4) and the on-chain `i128`/`Vec<Address>` types, and wrapping standard Soroban RPC simulate-then-submit flow. None of that is contract-layer work — it stays entirely in `warden-sdk`.

## Deferred, not forgotten
A `warden-stepup-signer` contract or mechanism — the actual second-factor signer that a smart wallet's `__check_auth` would require when `evaluate` returns `RequireStepUp` — is **not specified here**. Wiring `warden-contract` into a live passkey-kit smart wallet as a registered policy signer requires checking passkey-kit's current signer interface at build time; it should not be guessed or fabricated against an assumed API. `warden-app`'s v1 demo simulates the step-up confirmation as a second, distinct in-app confirmation step, which is enough to demonstrate the decision logic working end to end without depending on an unverified integration point.

## Next
This is the full spec for `warden-contract` and the target surface for `warden-sdk`. If this holds up, next is Phase 6: the standalone system prompt for your coding agent to actually build `warden-contract`.
