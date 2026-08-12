# HANDOVER — Condensation Calculator v0.5 / v0.5.1 (Compare Panels)

**Date:** 2026-08-12
**Repo:** StephanBK/condensation_calc · **Railway:** project `fortunate-dedication`, service `web`
**Live:** https://web-production-e8226.up.railway.app
**Commits this session:** c9938e7 (v0.5), 9aef0c2 (v0.5.1)

## What was built

Compare Panels mode: A/B comparison of two panels that differ ONLY in
cavity-side f-factor, under shared address/weather, indoor T, indoor RH,
and occupancy filter. Frontend-only feature — zero backend logic changes
(the frontend calls `/calculate` twice; sequential on purpose so call #1
warms the server weather cache and call #2 is near-instant).

### UI (static/index.html, single-file React via React.createElement, no build step)
- Mode toggle under header: "Single Panel | Compare Panels". Single mode untouched.
- Compare inputs (v0.5.1 order): Panel A (amber accent) and Panel B (teal accent)
  cards FIRST — free-text name + f-factor each (defaults 0.300 / 0.013) —
  then shared T/RH sliders below.
- Winner banner: navy block, giant "−XX%" in teal, "{name} wins", hr totals, Δ.
  Winner = fewer condensation hours (higher f always wins: T_surf = f·(T_in−T_out)+T_out).
- Side-by-side panel cards: big cond-hours number (risk-colored), days w/
  condensation, events, longest event, risk chip, "BEST" badge on winner.
- Delta strip: Δ hours / Δ days / Δ events / Δ longest ("{winner} advantage").
- **DiffHeatmap** (v0.5.1, replaced two stacked maps): ONE 365×24 map —
  red = both condense, amber = only A, teal = only B, gray = neither /
  excluded-by-filter. Legend shows hour counts. Winner's eliminated hours
  show as the loser's color.
- CompareDailyChart: outdoor + both surface temps + dew point line.
- Risk note blocks for both panels.
- PDF: same window.print() mechanism. Compare-specific letterhead title
  ("… — Panel Comparison"), comparison executive summary,
  `.cmp-avoid-break` print CSS keeps banner/cards whole.

### Key code artifacts
- `deriveResult(res, filter)` — module-scope shared per-panel derivation
  (extracted from single-mode useMemo; adds `condDays`). Both modes use it.
- `DiffHeatmap`, `CompareDailyChart` — new components.
- `Heatmap` gained optional `title` prop (unused now, kept for flexibility).
- App state: `mode`, `panelNameA/B`, `fA/fB`, `resultA/B`; `runComparison()`;
  memos `dA`, `dB`, `cmp` {winner, better, worse, deltaHours, pctReduction, tie}.

### Backend (main.py) — one change
- `/` now serves index.html with `Cache-Control: no-cache` → browser
  revalidates via ETag every visit; deploys appear immediately. Root cause of
  two "I still see the old version" incidents this session:
  1) plain browser cache (no header previously), 2) Chrome cache partitioning —
  the Odoo-iframe copy is cached under Odoo's top-level site, separate from the
  direct-URL copy. Both fixed one-time via hard refresh; header prevents recurrence.

## Testing
- `node --check` on extracted inline script (parse gate).
- Headless jsdom render test (`/tmp/render_test.js`, not committed): 12 checks —
  mount, toggle, compare inputs, banner, correct winner (higher f), BEST badge,
  −% display, diff heatmap + legend names, input order (cards above sliders),
  combined chart, delta strip, filter propagation, single-mode isolation.

## Open ideas / next steps
- System dropdown with WINDOW/THERM-calibrated f-factors (deferred since v0.3).
- Optional: 3+ panel comparison; cost overlay (condensation-hour deltas → $).
- SWR-over-VIG reference f ≈ 0.013–0.014 (confirmed in prior condensation memos).

## Ops notes
- Railway auto-deploys from GitHub main (~60–90 s).
- GitHub PAT used this session was pasted in chat → REVOKE it; get fresh from 1Password.
- Weather cache is in-memory → empties on every deploy; first lookup per
  location is slow (~5–10 s), then cached.
