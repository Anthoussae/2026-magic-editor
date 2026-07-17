# MSE2 Analysis: Overview

> Source analyzed: `/Users/yona/dev/james/MagicSetEditor2` (C++, wxWidgets, ~Windows-only builds in practice).

Magic Set Editor 2 lets users design custom cards for trading card games, render
them to images, print them, and export to HTML or play-testing formats. It also
provides set statistics and a booster-pack generator.

## The single most important architectural idea

**MSE2 is a generic, data-driven template engine — not a Magic-specific editor.**
The C++ core knows nothing about any particular card game. Games, card layouts,
mana-symbol fonts, and export formats are all *data packages* containing field
definitions, layout styles, images, and scripts in a small custom language. The
engine loads these and provides:

1. a schema system (Fields) with typed values and computed defaults,
2. a reactive script engine that keeps computed values up to date,
3. a WYSIWYG renderer/editor driven by stylesheet data,
4. undo/redo, persistence, import/export, statistics.

Any web reimplementation should preserve this split: an engine core + template
data, even if we ship only one built-in template initially.

## Source layout

| Directory | Contents |
|---|---|
| `src/data/` | Domain model: Set, Card, Game, StyleSheet, Field/Value/Style, Keyword, Symbol, actions (undo), import/export formats |
| `src/script/` | Scripting language: parser, bytecode VM, built-in functions, dependency tracking |
| `src/render/` | Rendering: card draw loop, per-field value viewers, rich text engine, symbol rasterizer |
| `src/gui/` | wxWidgets UI: main window panels, per-field value editors, symbol editor, dialogs |
| `src/gfx/` | Graphics utilities: resampling, masks, blending, combine modes, bezier math, generated-image node graph |
| `src/util/` | Reflection/serialization, package IO, action stack, rotation math |
| `src/cli/` | Command-line interface / script REPL |
| `doc/` | **Authoritative technical docs**: file formats (`doc/file/`), every data type (`doc/type/`), script language (`doc/script/`), function reference (`doc/function/`) |
| `data/` | Only `en.mse-locale` in the repo — game/style templates are distributed separately |

Note: the actual `magic.mse-game` / `magic-new.mse-style` packages are **not** in
the source repo; their formats are fully specified in `doc/type/*.txt`.

## Package ecosystem

Everything persistent is a *package* — a directory or zip with one main text data
file (custom tab-indented `key: value` format) plus assets:

- `.mse-game` — rules-side schema: fields, keywords, statistics, packs
- `.mse-style` — visual layout for a game's fields
- `.mse-symbol-font` — image font for inline symbols (mana costs)
- `.mse-export-template` — scripted HTML/text export
- `.mse-locale` — UI translations
- `.mse-include` — shared scripts/images
- `.mse-set` — a user's document: chosen game + stylesheet + cards + images

## User-facing feature inventory

- Live WYSIWYG card editor: in-place field editing, z-index layering, per-field
  rotation, non-rectangular masked fields
- Field types: text, choice, multiple-choice, boolean, color, image, symbol,
  package-choice, info label
- Rich tagged text: bold/italic, color/size/font runs, inline mana symbols,
  keyword atoms, bullets, justification; **auto-shrink text-to-fit** (binary-search
  font scaling); contour-aware text flow inside shaped boxes
- Spreadsheet-style card list: configurable/sortable columns, live filter
- Keywords: match patterns with typed parameters, auto-generated reminder text
- Statistics panel (bar/pie graphs, drill-down filtered lists)
- Random booster-pack generator (seeded)
- Vector symbol editor: bezier shapes, boolean combine ops, symmetry, gradients,
  image tracing
- Full undo/redo everywhere; find/replace; spellcheck; auto-replace rules
- Export: per-card images, templated HTML, Apprentice, Magic Workstation;
  print with page tiling
- Interactive script console; CLI with REPL and batch export
- Package manager with online updates; localization

## Companion docs

- [02-domain-model.md](02-domain-model.md) — entities, Field/Value/Style triad, undo, file format
- [03-scripting-and-templates.md](03-scripting-and-templates.md) — script language, template system, reactive dependency graph
- [04-rendering-and-ui.md](04-rendering-and-ui.md) — rendering pipeline, text engine, UI structure
