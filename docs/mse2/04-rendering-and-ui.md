# MSE2 Analysis: Rendering Pipeline & UI

> Source: `/Users/yona/dev/james/MagicSetEditor2` — `src/render/`, `src/gui/`, `src/gfx/`, `src/cli/`.

## 1. Card rendering (`src/render/card/`, `src/render/value/`)

**Viewer/editor split.** `DataViewer` draws a card to a device context and owns one
`ValueViewer` per visible field. `DataEditor`/`CardEditor` subclass it for editing.
Each field type provides a Viewer and an Editor (the editor composes rather than
inherits the viewer — deliberate diamond avoidance).

**Draw sequence** (`src/render/card/viewer.cpp`):
1. Set up a `RotatedDC` (quality: low / antialiased / subpixel).
2. Fill the stylesheet's `card_background`.
3. Run style scripts (positions, visibility, masks are all scriptable).
4. Two passes over viewers **sorted by `z_index`**: `prepare()` then `draw()` —
   painter's algorithm layering.

**Positioning.** A `Style` gives scriptable `left/top/width/height/right/bottom`
(any two per axis), `angle`, `visible`, `mask`, `z_index`, `tab_index`.
`Rotation`/`RotatedDC` (`src/util/rotation.hpp`) translate internal→external
coordinates with card-level zoom/rotation plus nested per-field rotation.

**Masks** do triple duty: alpha-clip field content, define the clickable hit-test
region, and provide per-row left/right bounds so text flows inside
non-rectangular shapes.

## 2. Rich text engine (`src/render/text/`) — the hardest subsystem

**Tagged text.** Values are strings with counted (nestable) SGML-like tags:
`<b> <i> <sym> <color:...> <size:...> <font:...>`, layout tags (`<line>`,
`<soft>`, `<li>`, `<margin:...>`, `<align:...>`), atoms (`<atom>`,
`<atom-param>` keyword params with a color palette), `<error>` (wavy underline).
Parsed into a `TextElement` tree (`src/render/text/element.cpp`).

**Symbol fonts.** A `SymbolFont` package maps text runs (`{W}`, `{U/R}`, `2`) to
images via longest-prefix matching; `always_symbol` styles auto-detect mana
symbols inline in normal text. Also provides the "insert symbol" editor menu.

**Layout** (`src/render/text/viewer.cpp`, ~870 lines): word-wrap honoring break
kinds (no/maybe/space/soft/hard/line), mask-contoured line bounds, paragraph
margins/bullets, vertical text, alignment incl. justify-by-words and
justify-by-characters, stretchable inter-line spacing.

**Auto-shrink to fit**: if content overflows, binary-search the largest font scale
that fits (with analytic bounds and previous-frame scale as a first guess).
Also horizontal stretch via draw-large-then-downsample.

The layouter doubles as the editing surface: `indexAt(point)`, cursor rects,
selection, scrolling.

## 3. Symbol editor (`src/gui/symbol/`, `src/render/symbol/`)

A small vector shape editor (mini-Inkscape) for set symbols:
- Model: tree of parts — bezier shapes (`ControlPoint`s with handle lock modes),
  groups, rotational/reflective symmetry; per-shape boolean **combine** modes
  (merge, subtract, intersect, difference, overlap, border).
- Rendering: rasterize to 1-bit border/interior buffers, boolean-combine, then a
  `SymbolFilter` colors inside/border/outside pixels (solid, linear/radial
  gradient). `SymbolVariation` = named filter+size, letting a stylesheet color
  one symbol differently per rarity.
- Tools: select/move/rotate/scale, point editing, basic shapes, symmetry; its own
  undo stack; image-to-symbol tracing.

## 4. Main UI structure (`src/gui/set/`)

`SetWindow` hosts exclusive panels behind a tab bar, each implementing a uniform
interface (clipboard, find/replace, card selection, own toolbar/menu):

- **Cards** — card list (sortable/configurable columns, filter box) + live
  WYSIWYG `CardEditor` with in-place per-field editing and tab-order navigation
- **Style** — stylesheet picker, data-driven styling options editor, live preview
- **Keywords** — keyword list + match/reminder/rules editors
- **Statistics** — bar/pie graphs + drill-down filtered card list
- **Set info** — set-level fields
- **Random pack** — seeded booster generation
- **Console** — interactive script REPL against the current card

All edits flow through `Action`s on a shared `ActionStack`; the UI and the script
manager both subscribe to the action stream.

## 5. Graphics utilities (`src/gfx/`)

Resampling (bilinear, sharpening, aspect-preserving), `AlphaMask` (hit-testing,
convex hull, per-row bounds), gradient/mask blending, ~24 Photoshop-style combine
modes, saturate/invert, rotate/flip, bezier math.

**`GeneratedImage` node graph** (`src/gfx/generated_image.hpp`): the scriptable
image-composition layer — a lazy, cacheable expression tree (blend, mask, crop,
enlarge, drop shadow, recolor, symbol-to-image...) exposed as script functions.
This is how template authors build card frames. Web equivalent: a canvas
compositing pipeline or layered CSS/SVG.

## 6. Export & CLI

- Image export per card (PNG/JPG, filename templates)
- HTML export via **export template packages** (scripted, not hardcoded)
- Apprentice and Magic Workstation exporters; MSE1 import
- Printing with page tiling (cards-per-page layout from physical sizes)
- CLI: batch export, symbol editor mode, interactive script REPL with raw
  machine-readable mode for driving MSE from other programs
- Background thumbnail-rendering thread for card lists

## Web reimplementation notes

- The z-ordered, style-driven draw loop maps naturally to either absolutely
  positioned DOM/SVG layers or a canvas renderer. Text-fitting (binary-search
  font scaling) and masked text flow are the hard parts; DOM/SVG gives
  measurement for free, canvas gives pixel fidelity for export.
- The viewer/editor split (render-only component + editing overlay) is worth
  keeping — it is also exactly what image export needs (render without editor).
- The symbol editor is a large, separable feature — a later phase; symbol fonts
  (images) are much cheaper and cover the visible 90% (mana symbols).
