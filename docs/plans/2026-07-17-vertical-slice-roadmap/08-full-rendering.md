# M8 — Full Template-Driven Card Rendering

- **ID**: M8 · **Size**: **lg — run `yona-plan` before implementing.** This
  file is a roadmap stub defining scope and exit criteria for that planning
  run.
- **Dependencies**: M3 (substrate proven), M6, M7. M5 provides the friendly
  template; local fixtures provide the compat targets.
- **ADR expectation**: possible (text-engine fidelity tradeoffs; image
  pipeline caching).

## Scope for the future planning run

Grow `mse-render` from the M3 skeleton into the real, style-driven renderer
(ported algorithms → GPL-3.0-or-later; re-audit the crate's file-level
licensing split from M3):

- Style-driven draw loop: z-index sorted, two-pass prepare/draw, per-field
  rotation, card background — `src/render/card/viewer.cpp`.
- Per-type value viewers: image (resample, mask, rotate), choice images,
  color fills, symbol-font text, text — `src/render/value/*`.
- `GeneratedImage` graph evaluation (blends, masks, crop/enlarge, drop
  shadow, recolor) on tiny-skia — `src/gfx/*`, `src/script/image.cpp`.
- Text engine on cosmic-text: tagged-text runs (`<b>`, `<i>`, `<color>`,
  `<sym>`, atoms), word-wrap with MSE2 break kinds, alignment,
  **shrink-to-fit binary search**, symbol-font inline rendering —
  `src/render/text/*`. Planned deferrals (record in ADR): contour/mask text
  flow, justify modes, vertical text — unless the chosen compat template
  needs them.
- Font handling: embedded fonts + a substitution map for common
  template-referenced fonts; document substitution behavior.
- **Golden-image testing** at two tiers: committed goldens for the demo
  template (CI); gitignored goldens exported from real MSE2 for community
  templates (local compat, skip-if-absent, with a documented perceptual-diff
  threshold — cross-rasterizer output won't be byte-identical).

## Exit criteria

1. Demo-template cards render correctly (CI goldens).
2. An agreed real community template renders its representative cards within
   the perceptual-diff threshold of MSE2's own export (local fixtures).
3. Render time per card acceptable in wasm (budget set during planning run).

## Agent reminders

Do not implement from this stub. Run `yona-plan` first. Reference analysis:
`docs/mse2/04-rendering-and-ui.md`. The text engine is the hardest part of
the whole port — plan it as its own sub-track.
