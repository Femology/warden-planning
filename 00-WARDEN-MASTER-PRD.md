# WARDEN — Master Project Brief (PRD)

**Read this file first, in full, before touching any other file or writing any code.**

This is the single source of truth for the Warden project. It defines what is being built, why, in what order, under what rules, and to what design standard. The detailed per-repo build prompts live in companion files referenced throughout — this file tells you which one to open, and when.

---

## 0. FILE MAP — WHAT'S IN THIS FOLDER

| File | What it is | When to read it |
|---|---|---|
| `00-WARDEN-MASTER-PRD.md` | **This file.** Project context, rules, build order, design system. | First, always. |
| `01-warden-phase4-scope-and-brand.md` | Scope, four-repo structure, what's explicitly out of scope. | Before starting any repo. |
| `02-warden-phase5-contract-architecture.md` | Full Soroban contract spec — data model, all 7 functions, events. | Before `warden-contract`; reference for all others. |
| `03-warden-phase6-contract-agent-prompt.md` | Standalone build prompt for `warden-contract`. | When building repo 1. |
| `04-warden-phase7-sdk-agent-prompt.md` | Standalone build prompt for `warden-sdk`. | When building repo 2. |
| `05-warden-phase7-app-agent-prompt.md` | Standalone build prompt for `warden-app`. | When building repo 3. |
| `06-warden-phase7-monitor-agent-prompt.md` | Standalone build prompt for `warden-monitor`. | When building repo 4. |

Each `03`–`06` file is written to be complete on its own. When you start a repo, that file is your working system prompt — but the design system in §6 of *this* file governs all UI in repos 3 and 4, and the rules in §4 govern everything.

---

## 1. WHAT WARDEN IS

Warden is an open-source **risk-policy engine for Stellar smart wallets**. It decides, per transaction, whether a transfer can proceed on the wallet's ordinary signature alone, or whether it needs a **step-up confirmation** (a second, explicit user confirmation) — based on three signals: the amount, whether the recipient is trusted, and how much the wallet has already spent in the last 24 hours.

**The one-paragraph version, for a non-technical reader:**
> Most wallet apps treat every payment the same. Send ₦500 to your sister and you get the same identity check as sending ₦5,000,000 to a stranger. Warden lets a wallet owner set their own rules — "under this amount, to people I've paid before, just let it through" — so the friction shows up where the risk actually is, instead of on every single tap.

### Why this problem is real
This isn't hypothetical. A live Nigerian fintech's public app-store reviews contain, verbatim, complaints like *"why do I have to scan my face for every single action on the app… there is no setting where you can change it. I had to move all my funds out immediately."* Users are abandoning products over undifferentiated authentication friction. Meanwhile the opposite failure — no friction on a large transfer to a fresh address — is how account-takeover losses happen. Warden is the missing middle.

### Why Stellar specifically (the load-bearing reason)
Soroban smart wallets support multiple signers with distinct roles evaluated inside the account's own auth check. That means the risk decision can live **on-chain, at the same trust boundary as the wallet itself**, rather than in a backend service that has to be separately operated and trusted. That's an architectural reason, not a "blockchain is fast and cheap" reason. Guard this argument — several decisions later in this brief exist only to protect it.

### Who it's for
Two audiences, and the product must serve both:
1. **Wallet owners** — set thresholds, manage trusted recipients, see where they stand against their daily limit.
2. **Integrating engineers at fintechs** — install an SDK, wire it into an existing wallet, and watch a dashboard showing whether their default policy is too strict or too loose.

---

## 2. THE FOUR REPOS

Built in this order. Each depends on the ones before it.

```
warden-contract  (Rust / Soroban)    → the policy engine. Decides.
       ↓
warden-sdk       (TypeScript)         → the client library. Builds & submits calls. Never signs.
       ↓
warden-app       (Next.js)            → reference wallet demo. Signs via passkey-kit.
       ↓
warden-monitor   (Node + Next.js)     → read-only telemetry. Watches. Never decides.
```

**The rule that governs the whole architecture:** the allow/step-up decision exists in exactly one place — `warden-contract`. The SDK relays it, the app acts on it, the monitor observes it. If any change would let the SDK, the app, or the monitor *make* or *override* that decision, reject the change. It would move the trust boundary off-chain and dissolve the reason the project exists.

---

## 3. HONEST LIMITATIONS — STATE THESE, NEVER HIDE THEM

Three real limitations exist in v1. They are deliberate, documented, and must be stated plainly in the relevant READMEs. Do not paper over them, and do not let marketing copy imply otherwise.

1. **The step-up gate is app-enforced, not cryptographically enforced.** `warden-app` asks the contract for a decision and its own UI refuses to proceed without confirmation — but nothing yet stops a modified client from ignoring the answer. Registering Warden as a true smart-wallet policy signer inside `__check_auth` is the natural next version and requires verifying passkey-kit's current multi-signer interface first. **Do not fabricate that interface.**
2. **The velocity window resets on expiry rather than sliding continuously.** A wallet could spend up to its cap just before a reset and again just after. Acceptable for v1; must be written up as a known limit.
3. **`warden-monitor`'s history can have gaps.** Soroban RPC retains events only for a limited recent ledger range. If the indexer falls behind that window, it must warn loudly rather than present partial data as complete.

---

## 4. NON-NEGOTIABLE WORKING RULES

These apply across all four repos, without exception.

### Git workflow
- **Never `git add .`.** Stage files by name, only after the initial scaffold commit.
- **One commit per logical unit** — one function, one type file, one test block.
- **Push immediately after every commit.** Never batch.
- **Conventional commits:** `type(scope): description` (`chore`, `feat`, `test`, `docs`, `fix`).
- Follow the numbered build sequence in each repo's own prompt file **in exact order**. It's dependency-ordered; skipping ahead creates bugs that look like regressions.

### Engineering discipline
- **No placeholders, stubs, `todo!()`, or "implement later" comments.** Every commit leaves working code.
- **No monetary value ever touches a float.** Not `f32`/`f64` in Rust, not JS `number`, not a SQL `REAL` column. Amounts are `i128` on-chain, exact decimal strings everywhere else. Convert only at the final display boundary.
- **Never guess an external API.** soroban-sdk, `@stellar/stellar-sdk`, passkey-kit, and the reference token contract all have surfaces that shift between versions. Verify against the actual pinned version's docs or types before writing against them. If something can't be verified, say so explicitly rather than inventing a plausible method name.
- **No `unwrap()` outside tests** (Rust). **No `any` used to dodge modeling a type** (TypeScript).
- **Every public function returns an explicit typed result.** Expected failures return typed errors — never panic, never throw a bare string.

### Scope discipline
- Build exactly what the spec says. Do not add an eighth contract function, a caching layer, a retry queue, an alerting system, multi-asset support, or an admin override. Each of these was considered and deliberately deferred — adding one silently is a scope violation, not initiative.
- If you believe something genuinely must be added, stop and say why before writing it.

---

## 5. TECH STACK SUMMARY

Full detail per repo lives in its own prompt file. Verify every version at build time.

| Repo | Stack | Notes |
|---|---|---|
| `warden-contract` | Rust, `soroban-sdk` **26.1.0** (not the 27.0.0-rc), Stellar CLI v27 | Verify Rust edition, MSRV, and wasm target triple against the SDK's own manifest — `wasm32-unknown-unknown` vs `wasm32v1-none` differs by SDK generation. |
| `warden-sdk` | TypeScript, ESM only, `@stellar/stellar-sdk`, vitest | Verify current published version of the Stellar SDK before pinning. |
| `warden-app` | Next.js (App Router), TypeScript strict, Tailwind, passkey-kit | Verify passkey-kit's package name and API before use. |
| `warden-monitor` | Node + SQLite indexer; Next.js + Tailwind dashboard | Polls Soroban RPC `getEvents` directly. No third-party indexer in v1. |

Network for v1: **Stellar Testnet only.**

---

## 6. DESIGN SYSTEM — UI & UX DIRECTION

This governs `warden-app` and `warden-monitor`. It is not optional styling advice; it is part of the spec.

### 6.1 The design problem, stated honestly

Warden's subject matter is **thresholds** — a line, and which side of it you're on. Everything visual should come from that, not from generic fintech or generic crypto.

The single most important conceptual point, and the one that should drive every color and motion decision:

> **Step-up is not failure.** It is not an error, not a rejection, not a warning. It is the system working correctly — friction arriving exactly where it should. A UI that paints step-up red teaches users that the product is punishing them.

So: **do not use a green/red pass/fail palette.** Allow and Step-up are two legitimate, healthy states. Red is reserved exclusively for actual errors (a failed transaction, an RPC timeout, invalid input). This one decision separates Warden from every generic dashboard.

### 6.2 Explicitly banned — these read as templated/AI-generated

Do not produce any of these. They appear regardless of subject matter and signal a lack of decision-making:
- Warm cream background (~`#F4F1EA`) + high-contrast serif display + terracotta accent (~`#D97757`).
- Near-black background with one acid-green or vermilion accent.
- The SaaS card kit: everything chopped into identical rounded cards, one border-radius on everything regardless of hierarchy, the same soft grey shadow under each, gradient washes as decoration.
- Tracked-out ALL-CAPS eyebrow labels above every heading.
- Meta strings joined with middle dots (`A · B · C`).
- `WORD — fragment` labels with a spaced em dash.
- A `→` appended to every link and button label.
- Tinted near-blacks (`#0B0B0B`, `#111`) standing in for black.
- Monospace used decoratively for small labels. *(Monospace for actual addresses and hashes is correct and expected — that's semantic, not decorative.)*
- Accenting a single word in a headline in a different color or italic.
- Numbered markers (`01 / 02 / 03`) on content that isn't genuinely a sequence.
- Fade-and-slide-up entrance animations on every section, and hover transitions on every card.

### 6.3 Palette

Base is a deep, genuinely blue ink — not a tinted black. The two decision states are distinguished by **temperature and weight**, not by good/bad color coding.

```
--ink-900   #101728   page background, deepest
--ink-800   #18213A   raised surface
--ink-700   #232F4F   borders, dividers, inactive track
--mist-100  #E9EDF7   primary text
--mist-400  #94A2C4   secondary text, labels

--clear     #2FD4A3   Allow state — cool, calm, low-energy jade
--gate      #FFB020   Step-up state — warm amber. Attention, not alarm.
--fault     #FF5A52   Errors ONLY. Never used for a step-up decision.
--edge      #6C5CE7   interactive accent (focus rings, active controls, links)
```

Light mode is **not required for v1**. Ship one confident dark theme rather than two mediocre ones. (If added later, the Allow/Step-up temperature relationship must survive the port — that's the load-bearing part, not the specific hexes.)

### 6.4 Typography

Two families, clearly distinct, neither of them the default reach:

- **Display / headings — `Bricolage Grotesque`** (Google Fonts, variable, has width and optical-size axes). Distinctive, slightly eccentric, holds up at large sizes. Use its variable axes deliberately — tighter width at large display sizes.
- **UI / body — `Instrument Sans`** (Google Fonts). Clean, excellent at small sizes, doesn't fight the display face.
- **Numerals — critical.** This product is entirely about amounts and thresholds. Use **tabular lining figures** (`font-variant-numeric: tabular-nums`) for every amount, balance, and limit so digits don't jitter as values change. Non-negotiable.
- **Addresses and hashes** — a monospace face (`JetBrains Mono` or system mono), truncated middle (`GABC…X7Q9`), full value on hover/copy.

Type scale: set a proper scale (roughly 1.25 ratio), don't hand-pick arbitrary px values. Body line length under 80 characters.

### 6.5 The hero moment — spend boldness here, nowhere else

`warden-app`'s landing page opens with **the threshold itself, live and draggable**: a horizontal band where the user drags an amount, and the decision flips in real time from Allow to Step-up as they cross their limit — the band's color, weight, and label transitioning with the drag. No copy explains it first; the interaction *is* the explanation.

This is the one place WebGL is justified: a subtle shader on the band — the surface behaving differently either side of the threshold (calm/settled below, agitated/energetic above). Everything else in both apps stays flat, fast, and disciplined.

**Do not put WebGL, particles, or shader effects anywhere else.** A policy engine's dashboard covered in effects reads as a portfolio piece, not infrastructure a fintech would trust with money.

### 6.6 Motion rules

- Motion that **answers a user action** — opening the step-up modal, a decision resolving, a value updating, a recipient being added — is welcome and should show *what changed*.
- Motion that **isn't user-triggered** is limited to the single hero moment above.
- The step-up modal deserves the most considered transition in the product: it should feel like a gate closing across the flow, not a generic dialog fading in. This is the emotional core of the whole product — the moment friction appears — and it should feel intentional and calm, never alarming.
- **Respect `prefers-reduced-motion`** everywhere, including the hero. Non-negotiable.

### 6.7 Logo direction

The mark should encode a **threshold, not a shield**. Shields, padlocks, and keys are the exhausted vocabulary of every security product and say "blocked" — the opposite of Warden's actual meaning.

Direction to explore: a single horizontal line with a break or a step in it — a gate in a wall, a bar that lifts. Works at 16px favicon size, works as a loading state (the step animating), works as the visual rhyme with the hero's threshold band. Wordmark set in Bricolage Grotesque at a tight width axis.

### 6.8 Writing in the UI

- **Plain language, from the user's perspective.** "You haven't sent to this recipient before," not "Recipient not in allowlist."
- **Active voice, and the action keeps its name through the whole flow.** The button says "Set limits" → the toast says "Limits set."
- **Errors explain what happened and what to do.** They don't apologize and they're never vague.
- **Empty states are invitations, not decoration.** A wallet with no policy yet gets a clear path to setting one, not a shrug illustration.
- Sentence case throughout. No filler.

### 6.9 Quality floor — required, not aspirational

- Fully responsive to mobile.
- Visible keyboard focus on every interactive element (use `--edge`).
- Every async action has a distinct loading state and a distinct error state. No silent failures.
- Color contrast meets WCAG AA against `--ink-900`.
- All modals are focus-trapped, dismissible via Escape, and return focus on close.

---

## 7. RECOMMENDED LIBRARIES

**Verify every package name and current version before installing** — several of these have renamed or relicensed recently. Do not install from memory.

**Motion & interaction**
- **Motion** (the library formerly published as Framer Motion) — primary animation layer for React. Confirm the current package name before adding.
- **GSAP** — for the hero's timeline-based choreography, where Motion's declarative model gets awkward. Its licensing changed recently in a more permissive direction; **verify current licensing terms yourself before depending on it commercially.**
- **Lenis** — smooth scroll, if the landing page needs it. Skip it in the dashboard.

**WebGL (hero only)**
- **Three.js** + **React Three Fiber** + **drei** — the standard, well-documented path.
- **OGL** — a much lighter alternative worth considering, since the hero needs one shader-driven band, not a 3D scene. Smaller bundle for a single effect.

**UI primitives & accessibility**
- **Radix UI** — unstyled, accessible primitives (dialog, popover, slider). The step-up modal and the threshold slider should be built on these so focus management and keyboard behavior are correct by default.
- **shadcn/ui** — copy-in components built on Radix + Tailwind. Useful as a starting point, **but restyle to this palette and type system.** Shipping shadcn defaults unchanged is exactly the templated look §6.2 bans.
- **Sonner** — toasts.
- **Lucide** — icons. Consistent, well-drawn, doesn't fight the type.

**Charts (`warden-monitor`)**
- **visx** (Airbnb) — composable, gives real control over how a chart looks. Preferred, since the dashboard's charts should match this design system rather than look like a chart library.
- **Recharts** — faster to ship, less control. Acceptable if time-constrained.

**Motion design & illustration**
- **Rive** — genuinely the right tool for interactive, state-driven illustration (e.g. the gate mark animating between states). Runtime is small and it responds to app state, unlike a static export.
- **Lottie** — for any linear animated sequence. Heavier than Rive for interactive states.
- **Avoid generic illustration packs** (unDraw and similar). They are instantly recognizable and will undercut everything else here. A small set of custom SVGs built from the threshold/gate motif is better than a large set of borrowed ones.

**Fonts**
- Google Fonts for Bricolage Grotesque and Instrument Sans. **Fontshare** is worth browsing if you want alternatives with more character — but change both faces together or not at all; mixing one distinctive face with one default undermines both.

**Inspiration to study, not to copy**
- **Awwwards / Godly / Land-book** for motion and layout reference. Study *how* they use restraint. Do not clone a template — the point of §6 is that Warden looks like Warden.

---

## 8. WHAT TO DO WHEN THINGS BREAK

Toolchain friction is expected, not exceptional.

1. **Diagnose before prescribing.** Read the actual error text. Trace to root cause (PATH conflict, version mismatch, DNS, disk) rather than applying a generic fix.
2. **Prefer surgical fixes over environment rewrites.** If a tool isn't found, fix the PATH or call it by full path — don't reinstall the toolchain.
3. **When multiple toolchain versions conflict** (e.g. two Rust installs), isolate the correct one explicitly in every command rather than trusting default resolution.
4. **When a version has a known incompatibility**, find the version that satisfies every constraint at once. Search for the actual constraint list rather than iterating version-by-version.
5. **If you start looping** — trying small workarounds without addressing root cause — stop and state the actual problem plainly instead of trying a sixth variation.

---

## 9. AFTER THE BUILD — REMAINING PHASES

Not part of the coding work, listed so the shape of the project is clear:

- **Phase 8 — Deployment.** Build and deploy `warden-contract` to Stellar Testnet; capture contract IDs; wire them into `warden-app` and `warden-monitor` environments.
- **Phase 9 — Hosting topology.** `warden-app` and the `warden-monitor` dashboard on a frontend platform (Vercel or equivalent). The `warden-monitor` indexer is a long-running stateful process with a database — it needs a platform built for that (Render or equivalent), not a serverless function. Co-locate the database with the indexer and use the internal connection string.
- **Phase 10 — Repo hygiene.** Branch protection with required status checks matching real CI job names; `CONTRIBUTING.md` and `SECURITY.md` (with an explicit "unaudited" disclaimer); README rewrite per approved-project conventions; GitHub topics; bulk issue generation via `gh` CLI; a `v0.1.0` release tag carrying the deployed contract addresses.
- **Phase 11 — Documentation site.** Covering both the non-technical user and the technical reviewer: what it is with real figures, the full decision lifecycle, complete contract reference, per-persona guides, developer setup, SDK reference with real code examples.
- **Phase 12 — Submission.** Live app URL, all four repo URLs, on-chain verification links, docs site, and a short demo video showing the three scenarios end to end. A submission without a working demo reads as unfinished.

---

## 10. START HERE

1. Read `01-warden-phase4-scope-and-brand.md` for scope and what's deliberately excluded.
2. Read `02-warden-phase5-contract-architecture.md` for the contract spec.
3. Open `03-warden-phase6-contract-agent-prompt.md` and build `warden-contract`, following its 26-commit sequence exactly.
4. Then `warden-sdk`, then `warden-app`, then `warden-monitor` — each with its own prompt file, each in dependency order.

Ask before deviating from a spec. Everything in these files was decided deliberately; if something looks wrong, it's worth a question rather than a silent fix.
