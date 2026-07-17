# M3 — Walking-Skeleton Card Render

- **ID**: M3 · **Size**: md · **Review gate**: none (spike-confirming ADR at end)
- **Dependencies**: M1; M2 (the render view should be a story).
- **ADR expectation**: expected (ADR: rendering substrate — confirms/refutes
  tiny-skia + cosmic-text + canvas-blit after real measurement).

## Scope

Prove the risky plumbing end-to-end with a **hardcoded** card: create
`crates/mse-render` (original code for now — AGPL until ported algorithms
arrive in M8, then re-examine file-by-file licensing) that draws a fixed
card layout (background, one image, styled text with an embedded font) via
**tiny-skia + cosmic-text** into an RGBA buffer; `app-web` blits it to an
HTML canvas; shown as a story and deployed.

Out of scope: MSE2 formats, scripts, templates, text shrink-to-fit, masks —
anything data-driven. This is a spike-shaped milestone with production
plumbing.

## Work items

1. `crates/mse-render`: `render_card(&CardSpec) -> Pixmap` where `CardSpec` is
   a hardcoded struct (title text, body text, a color, an embedded test
   image). tiny-skia drawing + cosmic-text layout with an embedded OFL font
   (e.g. Noto Sans; commit the font + its license file under `assets/fonts/`).
2. Host-native golden test: render → compare PNG against committed golden
   (pixel-exact; establish the golden-test helper used again in M8).
3. `app-web`: canvas element + blit of the RGBA buffer (wasm). Card view is a
   Dioxus component with a story showing 2–3 `CardSpec` variants.
4. Measure and record in the ADR: wasm binary size, render time, font
   rendering quality vs. native (screenshot comparison).
5. `docs/dev/rendering.md`: one page on the pipeline
   (engine renders pixels → app blits), why host-testable rendering matters.

## Validation

`just ci` green including golden test; story on deployed Pages shows the
card; ADR records measurements.

## Agent reminders

Do not commit unless asked. Keep `CardSpec` deliberately dumb — no premature
template abstractions. Stop and report if cosmic-text/wasm has blocking
issues (that's the point of the spike — the ADR may pivot the substrate).

## Definition of done

Rendered hardcoded card visible on the public Pages site; golden test in CI;
ADR 000x (rendering substrate) written with measurements.
