# Warden — Phase 8: Deployment

Deploys `warden-contract` to Stellar Testnet and captures everything `warden-sdk` and
`warden-monitor` will need in their environment config. This is a one-time, largely
irreversible sequence — a deploy or an `initialize` call cannot be undone, only
superseded by deploying again under a new contract ID. Read this file in full before
running Step 4.

## Prerequisites

- `warden-contract`'s 26-commit build sequence complete, `cargo test` green (confirmed:
  Phase 6, [github.com/Femology/warden-contract](https://github.com/Femology/warden-contract)).
- Stellar CLI installed and confirmed (confirmed: v27.0.0).
- `stellar network ls` includes `testnet` as a built-in network — no manual network
  config needed.

## Step 1 — Deployer identity

Generate a local identity and fund it via Testnet Friendbot in one step:

```bash
stellar keys generate deployer --network testnet --fund
stellar keys address deployer
```

This identity signs the deploy transaction and, in Step 5, acts as both `admin` and the
policy `wallet` for the one-time `initialize` call and the verify step. It is a
throwaway Testnet identity — not a production admin key, and not reused by `warden-app`.

## Step 2 — Build the release wasm

Already done in Phase 6's final commit, but re-run to build from a clean checkout if
needed:

```bash
cd contracts/warden
stellar contract build
```

Expected output: `warden.wasm` optimized to ~9KB, exporting exactly 7 functions
(confirmed in Phase 6).

## Step 3 — Reference asset

`initialize` requires a `reference_asset: Address` — the Soroban Asset Contract (SAC)
address for the token policy thresholds are denominated in. **Decision: use native XLM's
SAC for the v1 Testnet deployment**, not a custom-issued test USDC.

Reasoning, stated plainly so it can be revisited: the master PRD says "e.g. testnet
USDC" as an example, not a requirement, and `warden-contract` itself never reads or
validates this address — it's stored at `initialize` and never touched again by any of
the 7 functions (confirmed against the Phase 5 spec and the actual `lib.rs`). Native
XLM's SAC address is deterministic, requires no separate asset-issuing step, and is
trivial to verify on-chain. If a later phase (`warden-app`'s reference-asset display,
or a specific fintech-relevant token) needs a different asset, `reference_asset` is
reconfigurable by deploying `initialize` again under a new contract instance — it is
not load-bearing for the v1 demo.

Get the address:

```bash
stellar contract id asset --asset native --network testnet
```

## Step 4 — Deploy

```bash
stellar contract deploy \
  --wasm target/wasm32v1-none/release/warden.wasm \
  --source deployer \
  --network testnet \
  --alias warden
```

This prints the deployed **contract ID** (a `C...` address) and, in verbose mode
(`-v`), the transaction hash. Capture both.

**STOP HERE.** Show the contract ID before proceeding to Step 5 — `initialize` is
permanent for this contract instance; there is no `reset` function, by design (Phase 5
§ deliberately has no admin override).

## Step 5 — Initialize (permanent — do not run until Step 4's contract ID is confirmed)

```bash
stellar contract invoke \
  --id <CONTRACT_ID> \
  --source deployer \
  --network testnet \
  -- initialize \
  --admin <DEPLOYER_ADDRESS> \
  --reference_asset <NATIVE_SAC_ADDRESS>
```

## Step 6 — Verify

Confirm the contract is live and behaving correctly on a wallet that has never
configured a policy:

```bash
stellar contract invoke \
  --id <CONTRACT_ID> \
  --source deployer \
  --network testnet \
  -- get_policy \
  --wallet <DEPLOYER_ADDRESS>
```

Expected: a clean `PolicyNotFound` error (error code 3) — not a crash, not a decode
failure. This confirms the deployed contract's error path works exactly as the 24
passing unit tests predicted, against the real network this time.

## Step 7 — Capture the deploy ledger

`warden-monitor`'s indexer (Phase 7c) needs the ledger the contract was deployed at as
its starting point for `getEvents` polling — without it, it either re-scans from genesis
(slow, and may exceed the RPC's retention window) or misses early events. Get it from
the deploy transaction:

```bash
stellar tx fetch <DEPLOY_TX_HASH> --network testnet
```

The returned transaction result includes the ledger sequence it was included in. This
value is **`WARDEN_DEPLOY_LEDGER`** and cannot be recovered later except by knowing the
deploy transaction hash — write it down outside terminal scrollback.

## Env var block

Fill in with real values once Steps 4–7 succeed. This is the exact set `warden-sdk`,
`warden-app`, and `warden-monitor` will each need a subset of:

```bash
WARDEN_NETWORK=testnet
WARDEN_RPC_URL=https://soroban-testnet.stellar.org
WARDEN_NETWORK_PASSPHRASE="Test SDF Network ; September 2015"
WARDEN_CONTRACT_ID=
WARDEN_REFERENCE_ASSET=
WARDEN_DEPLOY_LEDGER=
WARDEN_DEPLOY_TX_HASH=
WARDEN_ADMIN_ADDRESS=
```

`WARDEN_RPC_URL` and `WARDEN_NETWORK_PASSPHRASE` are Stellar Testnet's standard,
publicly documented values, not specific to this deployment — worth a quick
cross-check against `stellar network ls --long` output at execution time in case they
have changed.
