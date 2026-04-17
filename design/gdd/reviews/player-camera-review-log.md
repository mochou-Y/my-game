# Player Camera — Review Log

## Review — 2026-04-17 — Verdict: APPROVED (after revision)

Scope signal: M
Specialists: game-designer, systems-designer, qa-lead, gameplay-programmer, creative-director
Blocking items: 5 (resolved) | Recommended: 5 (advisory)
Summary: Initial review identified 5 critical blocking issues including double sensitivity/invert_y application, soft stop direction bug, mouse delta-scaling error, and coordinate system errors. All blocking issues were resolved through cross-document edits to both Player Camera and Input Handling GDDs. Design now has clear ownership: Input Handling owns sensitivity and inversion; Player Camera applies pre-processed deltas.
Prior verdict resolved: First review

### Blocking Issues Resolved

| # | Issue | Resolution |
|---|-------|------------|
| 1 | Double sensitivity application (square of intended) | Input Handling now owns sensitivity; Player Camera receives pre-scaled values |
| 2 | Double invert_y application (double negation) | Input Handling now owns inversion; Player Camera receives pre-inverted values |
| 3 | Soft stop dampens movement away from limit | Formula rewritten with direction check; only dampens toward limit |
| 4 | Mouse incorrectly delta-scaled | Input Handling Rule 3 clarified: mouse NOT scaled, gamepad IS scaled |
| 5 | AC 19-20 coordinate system error (+Z vs -Z) | Corrected to Godot's -Z forward; added tolerances |

### Advisory Items (not blocking)

1. Soft stop contradicts Player Fantasy ("without resistance") — accepted as design decision
2. Pitch limit 85° may block ceiling puzzles — accepted, can adjust later if needed
3. Missing motion sickness/accessibility considerations — advisory for future iteration
4. Camera hierarchy not specified — implementation detail
5. 1:1 raw input design choice — intentional, consider optional smoothing later

### Files Modified

- `design/gdd/player-camera.md` — formulas, ACs, dependencies, edge cases
- `design/gdd/input-handling.md` — Rule 3, formulas, edge cases, contracts
- `design/gdd/systems-index.md` — status updated to Approved
