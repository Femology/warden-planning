# Warden — Design System v2 & Site Architecture

**This supersedes section 6 of `00-WARDEN-MASTER-PRD.md` and the palette in `09-warden-ui-fix-and-redesign-prompt.md`.** The base color family changed (blue-ink → forest-black) — if any of that palette is already built, it needs a real re-skin, not incremental edits. Give this file to Claude Code as the authoritative design reference from here on.

---

## Decisions locked this round

- **No email/password accounts, ever.** Wallet connection (Freighter primary, passkey-kit secondary) remains the only identity. Email is one optional field for state-change notifications — see §7. No login flow, no session system, no password reset flow to build.
- **Both dark and light themes ship in v1.** Both need to hold up to the same quality floor from the original design system (§6.9 of the master PRD) — don't ship a half-finished light mode.

---

## 1. Full sitemap

**Marketing:**
- `/` — landing (see §5)
- `/how-it-works` — grade 5–7 English, its own page
- `/security` — the trust page: escalate-easy/de-escalate-hard model explained, link to SECURITY.md, an honest unaudited-software statement
- `/developers` — quickstart, SDK, contract reference, events, examples (per the existing Phase 7-app-era spec, now surfaced as marketing too)
- `/protocol` or `/about` — plain-English three-layer story, why it's open source
- `/privacy`, `/terms`, `/risk-disclosure` — see §8

**App** (unchanged from the earlier frontend spec): `/app`, `/app/state`, `/app/transactions`, `/app/transactions/new`, `/app/policy`, `/app/recipients`, `/app/guardians`, `/app/recovery`, `/app/risk`, `/app/audit`, `/app/settings`, `/app/onboarding`.

---

## 2. Palette v2

### Dark (default)
```
--ink-900   #0D1712   base — deep forest-black, not blue-black
--ink-800   #16251E   raised surface
--ink-700   #223229   borders, dividers, inactive track
--mist-100  #EAF2ED   primary text
--mist-400  #93A99C   secondary text, labels

--clear     #22C38D   Allow / calm state — canopy green
--gate      #F2994A   Step-up state — ember-gold, warm not alarming
--fault     #FF5A52   Errors ONLY, unchanged rule — never a policy decision
--edge      #6C5CE7   interactive accent — unchanged
```

### Light
```
--ink-900   #FBF8F1   base — warm bone/parchment, explicitly not a blue-family neutral
--ink-800   #F2EDE0   raised surface
--ink-700   #DCD3BE   borders, dividers
--mist-100  #1D2A22   primary text (dark forest-green-black, not pure black)
--mist-400  #5C6B60   secondary text

--clear     #1C9C73   Allow — deepened for light-background contrast
--gate      #D97B2C   Step-up — deepened ember for AA contrast on light
--fault     #D93B33   Errors — deepened red for light background
--edge      #5A4BC4   interactive accent — deepened
```

**Both themes get independently checked against WCAG AA before either is considered done** — deepening the accent colors for light mode isn't optional, a light-mode-specific contrast pass is required, not a straight reuse of the dark values.

### Where the "fire" actually lives
The ember/volcanic feeling is concentrated in exactly one place: the landing hero's WebGL shader background (paper-design/shaders, mesh gradient, slow ember-glow in `--gate`/`--clear` tones behind the threshold drag). It does not spread into cards, buttons, or dashboard chrome — those stay disciplined and flat so the product still reads as trustworthy infrastructure, not a themed landing page.

---

## 3. Typography — unchanged, carried forward
Bricolage Grotesque (display), Instrument Sans (body), tabular numerals for every amount, monospace truncated addresses. No changes here — this survived the palette pivot intact.

---

## 4. Trend application map — where each idea actually goes, and where it doesn't

| Trend | Verdict | Where it lives |
|---|---|---|
| Bento Box | **Full adopt** | `/app` overview, `/app/audit`, the monitor dashboard — genuinely the right pattern for dense financial data |
| Glassmorphism | **Adopt, scoped** | Header and footer chrome only — frosted, blurred. Never on content cards holding real data |
| Ethereal | **Adopt, scoped** | The quality of the hero's shader glow specifically — soft, mist-like rendering. Not the UI's general visual weight |
| Surrealism | **A touch, one place** | One illustration only (the step-up moment or an empty state) gets one dreamlike compositional choice — the gate in ambiguous space. Not a running visual language |
| Neumorphism | **Dropped** | Its signature low element/background contrast conflicts directly with the WCAG AA requirement already locked into this project. Borrow only "soft card depth," not the low-contrast look |
| Skeuomorphism | **Dropped, already present elsewhere** | The gate/threshold motif is already the one real-world metaphor this product needs — no literal textures beyond that |
| 3D carousel | **One place only** | The open-source section's five-repo showcase on the landing page. Never in the authenticated app |

---

## 5. Landing page — full section spec

**Header:** logo mark + wordmark, nav (How it works / Security / Developers), Connect Wallet button, theme toggle. Glassmorphism treatment per §4.

**Hero:** the draggable threshold band from the original design system, now rendered against the ember-glow shader described in §2. Headline direction: state the mechanism plainly — *"Most wallets treat every payment the same. Warden doesn't."* — no unprovable claims about fraud prevention.

**Section 1 — The problem.** The $5-to-a-known-recipient vs. $5,000-to-a-stranger comparison, one glance, minimal copy. Copywriting rule applied here specifically: lead with the reader's own frustration, not with Warden's features.

**Section 2 — How it works.** The three-layer flow (signal → policy → enforcement) and, load-bearing, the explicit statement that intelligence never controls funds — this is the single most important trust claim in the whole site and belongs here, not buried later.

**Section 3 — The security model.** The state-machine visual (`NORMAL → WATCH → RESTRICTED → CHALLENGED → FROZEN`), with escalate-easy/de-escalate-hard shown as a visual asymmetry, not just stated in text.

**Section 4 — Open source.** The five repos (`contract`, `sdk`, `app`, `monitor`, and `oracle` if it exists by launch) shown as the one 3D carousel/infinite-showcase moment, each linking to a real GitHub URL. Never fabricate stars or activity — if a repo is new and has zero stars, show zero, don't decorate it.

**Footer:** glassmorphism per §4, links to all marketing pages, the legal pages from §8, and social/GitHub.

---

## 6. Icons and illustration sourcing

**Icons:** Lucide (base, per the original spec) plus **Phosphor Icons** (`phosphor-icons/react`) and **Tabler Icons** (`tabler/tabler-icons`) — both real, open-source, line-art families that sit well with the gate motif rather than fighting it.

**Abstract/organic texture, not stock illustration:** **Haikei** (`app.haikei.app`) for unique generated SVG blob/gradient textures — confirm it's still live before depending on it, don't assume. The pattern generators already identified in earlier research (`jasonlong/geo_pattern`, Pattern Monster) for subtle canopy-like generative backgrounds. **No verified specific library exists for literal leaf/nature line-art** the way one exists for the gate motif — say so honestly rather than pointing at something unconfirmed; build any nature-adjacent line-art custom, sparingly, from the same simple geometric discipline as the gate mark itself.

---

## 7. Email notification feature (resolved scope)

```
Add ONE optional field, reachable from /app/settings: an email address, saved
purely as a notification destination. No password, no login, no session tied to
it — the wallet connection remains the only way into the app. Sending a
notification when the wallet's on-chain state changes (WATCH/RESTRICTED/
CHALLENGED/FROZEN, and a recovery proposal being raised) requires a small backend
service that watches the same events warden-monitor already indexes and sends an
email — it does not gate access to anything, and losing access to that email
address never blocks using the wallet itself.

State this scope explicitly in the settings UI copy: "We'll only use this to
alert you if your wallet's security state changes. It's never required, and
removing it never affects your wallet."
```

---

## 8. Legal pages — draft direction, not final text

- **Terms of Use** — standard open-source software terms, no warranty, no liability for fund loss, licensed under whatever license the repos use.
- **Privacy Notice** — scoped to what's actually collected: an optional email address for notifications, and nothing else server-side (everything else is on-chain, public, and not "collected" in the traditional sense). Say this plainly rather than padding with generic clauses that don't apply.
- **Risk Disclosure** — its own page, not a Terms footnote: this is unaudited software, it makes real decisions about real funds, self-custody means no one can reverse a mistake, and the escalate/de-escalate model is explained again in plain terms here.

**All three need an actual lawyer's review before publishing** — this is direction for the copy, not something to ship as final legal text on its own authority.

---

## 9. Motion map

- Staggered/sequential reveal on scroll — yes, throughout the landing page.
- Hover states on every interactive element — yes, standard.
- One looping idle animation — the gate mark, per the earlier Rive-based logo spec, not decorative motion elsewhere.
- Scale/zoom on focus — yes, in moderation, for drawing attention to the current security state specifically.
- 3D carousel — confined to §5's open-source section, nowhere else.
- `prefers-reduced-motion` respected everywhere, including all of the above — unchanged rule from the original system.
