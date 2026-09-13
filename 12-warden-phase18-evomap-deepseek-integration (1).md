# Warden — Phase 18 Integration: Evomap (DeepSeek V4.1 Flash) for the Explanation Feature

This supersedes the generic Phase 18 prompt in `11-warden-feature-roadmap-phases-14-19.md` now that a real provider is confirmed. **It covers the explanation feature only.** Do not use this integration as the source of a risk score for Phase 17 without a separate, explicit decision — see the boundary note at the end.

---

## Provider

**Evomap**, OpenAI-SDK-compatible Chat Completions endpoint, model DeepSeek V4.1 Flash.

```
Base URL:  https://api.evomap.ai/v1
Auth:      Authorization: Bearer sk-evomap-<key>
```

**The API key is never written into any file, prompt, or commit.** It is read only from an environment variable (`EVOMAP_API_KEY`) at runtime. The key that was shared in chat has been exposed and must be rotated on Evomap's dashboard before this is wired up — confirm that's done before writing any code against it.

## Model string — confirmed

Evomap's gateway exposes this model as **`evomap-deepseek-v4-flash`** (confirmed directly from a working request against their endpoint — not the bare `deepseek-v4-flash` name from DeepSeek's own docs, which is a different string). Use exactly this value. If Evomap ever changes their catalog, that will surface as an explicit "model not found" error, not a silent failure — treat that error as a signal to re-check their current model list, not something to work around.

## Local setup — the actual first step, before any code

```
Add .env.local to .gitignore in the FIRST scaffold commit, before anything else
touches this repo — not as a later cleanup step. Then create .env.local
(untracked) containing:

EVOMAP_API_KEY=<the rotated key, pasted here and nowhere else>

The server-side helper below reads process.env.EVOMAP_API_KEY. Confirm
.env.local is actually excluded by running `git status` after creating it —
it must not appear as a trackable file. If it does, the .gitignore entry is
wrong; fix that before writing another line of code.
```

## Integration

```
Add a small server-side helper (never call this from the browser directly — the
key must never reach client-side code) that wraps the OpenAI SDK pointed at
Evomap's base URL, reading EVOMAP_API_KEY from the environment.

This is used ONLY by the "Explain this" feature from Phase 18. Reconfirm every
constraint from that phase's spec still applies exactly as written:

- Input to the model is ONLY the structured on-chain facts for one event
  (previous state, new state, the specific reason code or policy rule that
  triggered it) — nothing else, no raw user data, no conversation history, no
  access to call any Warden function.
- Output must be validated against the fixed schema (summary, factors, next
  steps) before being rendered. If the response doesn't match the schema, or
  references a reason code that wasn't in the input, discard it and show the
  pre-written fallback explanation for that reason code instead.
- The model has no ability to change state, approve a transaction, or influence
  anything on-chain, directly or indirectly. It only produces text for display.

Test explicitly: a malformed or off-schema response from Evomap falls back
correctly. A response referencing a reason code not present in the input is
discarded, not rendered. The API key is confirmed absent from every committed
file (grep the repo for the key prefix before considering this done).
```

## The boundary — do not cross this without a separate decision

This integration answers "how do we explain a decision that's already been made." It does **not** answer "how do we decide what the risk score is" — that's Phase 17, and it remains unresolved. Using this same Evomap/DeepSeek connection to *generate* a risk score that feeds escalation would mean an LLM's non-deterministic, promptable output is what moves a wallet toward `RESTRICTED` or `FROZEN` — a materially weaker and more attackable foundation than either the deterministic rules already built (Phase 14/15) or a real risk vendor with actual behavioral data. If that's genuinely the intent, it needs its own explicit spec and its own security review — it does not fall out of this integration for free, and this file does not authorize it.
