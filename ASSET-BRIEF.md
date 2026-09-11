# Warden — Visual Asset Brief

Every asset Warden's redesign needs, with the exact filename to save it as and a
detailed description to source or commission against. Drop finished files into the
folders noted per section. SVG preferred everywhere except photographic/3D renders
(PNG/WebP, noted explicitly).

Palette to hold every asset to (from the design system):
```
--ink-900   #101728   page background, deepest
--ink-800   #18213A   raised surface
--ink-700   #232F4F   borders, dividers
--mist-100  #E9EDF7   primary text/light linework
--mist-400  #94A2C4   secondary text/muted linework
--clear     #2FD4A3   Allow state — cool jade
--gate      #FFB020   Step-up state — warm amber
--fault     #FF5A52   Errors only
--edge      #6C5CE7   interactive accent
```

Fonts in any asset with type baked in: **Bricolage Grotesque** (display) / **Instrument
Sans** (body).

---

## A. Logo & brand marks

Folder: `warden-app/public/logo/`

Direction: a threshold or gate — a line with a break or step in it. **Not** a shield,
padlock, or key (that vocabulary means "blocked," which is the opposite of what Warden
does). Think Cash App's confident geometric marks or Betterment's simple wordmark-led
identity — not a literal financial icon.

1. **`logo-mark-01-break.svg`** — Concept 1: a single horizontal bar with a visible gap
   partway across it, the two halves offset slightly in weight or position — like a
   circuit breaking and reconnecting at a different height.
2. **`logo-mark-02-gatepost.svg`** — Concept 2: a horizontal line crossing a vertical
   post, with a gap in the horizontal line right where it meets the post — reads as a
   gate arm mid-lift.
3. **`logo-mark-03-dashed-solid.svg`** — Concept 3: two horizontal bars of different
   weight — one solid, one dashed — meeting at a shared point on the left, fanning
   apart slightly to the right. Solid = allowed path, dashed = the step-up path.
4. **`logo-mark-04-step.svg`** — Concept 4: a single continuous line that steps up like
   a stair — flat, rise, flat — rendered as one unbroken path, no corners rounded.
5. **`logo-favicon-16.svg`** (and `-32.svg`) — Whichever mark concept #1–4 is chosen,
   simplified until it reads clearly at 16×16 and 32×32px. Usually means thickening
   strokes and removing any fine detail.
6. **`logo-lockup-light.svg`** / **`logo-lockup-dark.svg`** — The chosen mark next to
   the wordmark "Warden" set in Bricolage Grotesque, tight width axis. Light version
   for light surfaces, dark for `--ink-900` backgrounds.
7. **`logo-animated-gate.json`** (Lottie) or **`.riv`** (Rive) — The chosen mark
   animating between closed and open states via a simple stroke-dasharray reveal or
   path morph. Reused as: page-load spinner, and the opening beat of the step-up
   modal's transition.

---

## B. Landing page (marketing, `/`)

Folder: `warden-app/public/illustrations/landing/`

Direction: closer to Stripe's confident gradient-mesh abstraction or Cash App's bold
flat-geometric props than Betterment's soft storybook illustration — Warden's subject
(thresholds, limits, velocity) suits sharper, more graphic shapes over rounded/friendly
ones. Dark-mode-first (the whole product ships one dark theme).

8. **`hero-threshold-band-idle.svg`** — Static fallback/poster frame for the hero's
   live draggable threshold band (the real one is WebGL/interactive — this is what
   shows before it hydrates, or for `prefers-reduced-motion` users). A horizontal band
   with a visible midpoint marker, jade-colored below the line, amber above it.
9. **`section-amount.svg`** — Small supporting graphic for the "amount" explainer
   section: two bars of different length against a shared baseline, one jade one
   amber, no numbers/labels baked in (real data overlays in code).
10. **`section-recipient.svg`** — Supporting graphic for the "trusted recipient"
    explainer: two small circular nodes connected by a solid line (trusted) and a third
    node connected by a dashed line (untrusted/new).
11. **`section-velocity.svg`** — Supporting graphic for the "velocity/24h window"
    explainer: a simple arc or bar filling left-to-right, with a marked threshold point
    partway along, echoing the hero band's visual language at smaller scale.
12. **`bg-texture-noise.png`** — Optional: a very subtle grain/noise texture overlay
    (2–4% opacity) for `--ink-900` sections, the way Clyde's hero uses soft radial glow
    to add depth without literal imagery. Skip if it reads as noise rather than depth.
13. **`og-share-image.png`** (1200×630) — Social preview card: wordmark + a graphic
    treatment of the threshold band, for link unfurls on X/Slack/etc.

---

## C. App — empty states

Folder: `warden-app/public/illustrations/empty-states/`

Direction: simple, geometric, line-work only — these are functional moments, not
marketing beats, so keep them quieter than the landing page. Single accent color max
per illustration (jade, amber, or the edge purple — never combine).

14. **`empty-no-policy.svg`** — Shown on first visit to `/app/policy` before any policy
    is set. A single incomplete threshold band (like the hero's, but static and
    "unset" — a dashed outline where the marker should be, not yet placed).
15. **`empty-no-trusted-recipients.svg`** — Shown on `/app/policy` when the trusted
    list is empty. A single node with a dashed, empty ring around it — "add the first
    one" energy, not a sad/broken feeling.
16. **`empty-no-transfer-history.svg`** — Shown on `/app/transfer` or `/app/velocity`
    before any evaluation has happened. A flat horizontal line with no marks on it yet
    — the velocity bar with nothing recorded.
17. **`empty-wallet-not-connected.svg`** — Shown on `/` (or `/app`) before a wallet is
    connected at all. The gate-mark logo (once chosen from section A) in an "at rest"
    open state, inviting rather than blocking.

---

## D. App — key moments

Folder: `warden-app/public/illustrations/moments/`

18. **`stepup-moment.svg`** — Sits inside `StepUpConfirmModal`. This is the emotional
    core of the product — reinforce "this is the system working correctly," not an
    alarm. Use the gate mark mid-close (not fully shut), amber, calm linework. No
    exclamation marks, no red, no hazard imagery.
19. **`stepup-reason-amount.svg`** — Small icon-scale accent next to the "amount
    exceeded" copy in the modal — a single bar crossing a threshold line.
20. **`stepup-reason-recipient.svg`** — Small icon-scale accent for "new recipient" —
    a single node with a dashed ring, echoing #15.
21. **`stepup-reason-velocity.svg`** — Small icon-scale accent for "velocity exceeded"
    — a filled bar overrunning a threshold mark, echoing #11.
22. **`success-confirmed.svg`** — Shown after a transfer completes (allowed or
    confirmed step-up). The gate mark fully open, jade, brief and understated — this
    is not a confetti moment, it's a receipt.
23. **`error-generic.svg`** — For genuine errors (RPC timeout, failed transaction) —
    the *only* place `--fault` red appears. Keep it distinct from every step-up
    illustration above so users never confuse "step-up" with "broken."

---

## E. `warden-monitor` dashboard

Folder: `warden-monitor/dashboard/public/illustrations/`

Same visual language as C/D, reused rather than reinvented — the dashboard should look
like it belongs to the same product, not a generic admin panel (Stripe's dashboard
screenshot you shared is the right reference for information density; not its color
treatment).

24. **`empty-no-events-indexed.svg`** — Shown on the summary page before the indexer
    has caught anything. A flat, empty version of the reason-breakdown bars.
25. **`empty-wallet-no-history.svg`** — Shown on a wallet drill-down page for an
    address with no evaluation history yet — reuse `empty-no-transfer-history.svg`
    from section C directly rather than making a new one.
26. **`empty-wallet-no-policy.svg`** — Shown on a wallet drill-down page for an address
    that's never called `set_policy` — reuse `empty-no-policy.svg` from section C.

---

## F. Utility / meta

Folder: repo root and `public/` as noted.

27. **`favicon.ico`** (multi-size: 16/32/48) — Derived from the chosen logo mark, for
    browsers that don't support SVG favicons.
28. **`apple-touch-icon.png`** (180×180) — Same mark, filled square background at
    `--ink-900`, for iOS home-screen saves.
29. **`warden-docs/public/favicon.svg`** — Same mark as A, reused for the GitBook site
    once GitBook's custom-favicon option is set up (already has a placeholder there
    from an earlier pass — replace it once the real mark is chosen).

---

## What to send back

For each numbered item: the finished file, saved under the exact filename above, in
the folder noted for its section. If you're commissioning rather than generating —
this same list works as a brief to hand a designer as-is.

Nothing here is final until you've picked which of the four logo concepts (A.1–4) to
run with — everything downstream (favicon, animated mark, empty-wallet illustration,
step-up moment) derives from that choice, so start there.
