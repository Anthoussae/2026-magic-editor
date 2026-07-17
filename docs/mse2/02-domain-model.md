# MSE2 Analysis: Core Domain Model

> Source: `/Users/yona/dev/james/MagicSetEditor2` — `src/data/`, `src/util/`.

MSE2's domain layer is cleanly separated from its wxWidgets UI. It is driven by a
reflection-based (de)serialization system plus the scripting/dependency engine
(see [03-scripting-and-templates.md](03-scripting-and-templates.md)).

## 1. Entity map

```
Game (defines fields) ◄──── StyleSheet (defines styles for those fields)
   ▲                              ▲
   │ game                         │ stylesheet (default)
   └──────────── Set ─────────────┘
                  │ cards[]           each Card may override stylesheet/styling
                  ▼
                Card ── data: IndexMap<Field,Value>  (indices align with Game.card_fields)

Field ──newValue()──► Value      (Value.fieldP → Field)
Field ──newStyle()──► Style      (Style.fieldP → Field; StyleSheet stores these)
```

### Package (`src/util/io/package.hpp`)
Everything persistent is a *package*: a directory **or** zip file containing one main
text data file (named `set`, `game`, `stylesheet`, ...) plus resource files (images,
symbols). Packages carry `version`, `full_name`, `icon`, and versioned `dependencies`
on other packages. Writes are staged and committed on save; files no longer referenced
are garbage-collected.

### Set (`src/data/set.hpp`)
The top-level user document (`.mse-set`):
- `game` (required) and default `stylesheet` (must belong to that game)
- `data` — set-level field values (aligned with `game->set_fields`)
- `cards` — the card list
- `keywords` — set-specific keywords (added to the game's)
- `styling_data` — per-stylesheet styling option values
- `pack_types` — booster pack definitions
- `actions` — the undo/redo stack; a `SetScriptManager` listens to it to drive
  reactive script updates

### Card (`src/data/card.hpp`)
- `data` — field values aligned with `game->card_fields`
- optional per-card `stylesheet` override + per-card styling data
- `notes`, `time_created`, `time_modified`

### Game (`src/data/game.hpp`) — the *rules-side schema* (`.mse-game`)
- `set_fields`, `card_fields` — the Field definitions
- `init_script` — variables/functions available to all scripts
- keyword machinery (`keywords`, `keyword_parameter_types`, `keyword_modes`)
- `statistics_dimensions` / `statistics_categories`, `pack_types`, `word_lists`
  (autocomplete), `auto_replaces`

### StyleSheet (`src/data/stylesheet.hpp`) — the *look* (`.mse-style`)
- targets one `game`; `card_width/height/dpi`, `card_background`
- `card_style` / `set_info_style` — one Style per game Field
- `extra_card_fields` + styles — decorative fields the stylesheet adds itself
  (frames, boxes)
- `styling_fields` — user-adjustable styling options (data-driven options UI)

### Keyword (`src/data/keyword.hpp`)
Parameterized reusable rules text: `keyword`, `match` pattern with typed
`<atom-param>` parameters, `reminder` script, `mode`. A `KeywordDatabase` trie
expands keywords inside card text.

### Symbol (`src/data/symbol.hpp`)
Vector graphics (bezier shapes, groups, symmetries) edited in the built-in symbol
editor; own `ActionStack`. Distinct from `SymbolFont` (image font for mana symbols).

### Statistics & Packs (`src/data/statistics.hpp`, `src/data/pack.hpp`)
`StatsDimension` (a graph axis, often auto-derived from a field, or scripted) and
`StatsCategory` (a chart). `PackType`/`PackItem` describe booster recipes; a
`PackGenerator` samples cards randomly.

### ExportTemplate (`src/data/export_template.hpp`)
A package holding an export script plus its own option Fields/Styles (data-driven
export options dialog). Format handlers for HTML, MSE1, MWS, Apprentice, clipboard
live in `src/data/format/`.

## 2. The Field / Value / Style triad — the heart of the model

All three carry a back-reference to the Field; they line up positionally via a shared
`Field::index` across three parallel `IndexMap`s (`Game.card_fields`, `Card.data`,
`StyleSheet.card_style`).

- **Field** (`src/data/field.hpp:40`) — the *schema*: name, caption, editability,
  whether saved, card-list/statistics config, sort script. Factory methods
  `newValue()` / `newStyle()`.
- **Value** — one *instance's data* on one card/set: `clone()`, `toString()`,
  `update(Context&)` (runs scripts), age stamp for reactive updates. This is what gets
  saved per card.
- **Style** — *display/layout*: scriptable `left/top/width/height/right/bottom`,
  `angle`, `visible`, `mask`, `z_index`, `tab_index`; factory methods `makeViewer()` /
  `makeEditor()` (per-field-type rendering and editing widgets).

### Field types (`src/data/field/*`)

| `type:` | Value data | Notes |
|---|---|---|
| `text` | tagged string, defaultable | `script:` post-processing, `default:` computed value, multiline |
| `choice` | string, defaultable | tree of choices with groups, per-choice images/colors, render as text/image/checklist |
| `multiple choice` | `"a, b, c"` string | subclass of choice |
| `boolean` | yes/no | subclass of choice |
| `image` | local filename | image stored in the set package |
| `symbol` | local filename | references a `.mse-symbol` |
| `color` | color, defaultable | named choices + optional custom |
| `info` | string | non-editable label |
| `package choice` | package name | e.g. pick a symbol font |

`Defaultable<T>` (`src/util/defaultable.hpp`) wraps a value plus an "is this the
script-computed default or a user override?" flag — central to how computed defaults
coexist with manual edits.

## 3. Undo/redo — command pattern (`src/util/action_stack.hpp`, `src/data/action/`)

- `Action`: one `perform(to_undo)` method that both does and undoes (called with the
  flag alternating); optional `merge()` coalesces consecutive typing.
- `ActionStack`: undo/redo vectors, save-point tracking, and an observer list —
  both the UI (`SetView`) and the `SetScriptManager` subscribe to actions.
- Catalogue: value edits (per field type, with text-selection-aware merge), add/reorder
  cards, change stylesheet, keyword edits, symbol editor actions, and a generic
  `GenericAddAction<T>` primitive for list add/remove.
- `ScriptValueEvent`/`ScriptStyleEvent` are *notifications* (not undoable) broadcast
  when scripts recompute values, so viewers repaint.

## 4. File format (`src/util/io/reader.cpp`, `writer.cpp`, `reflect.hpp`)

- Tab-indented, line-oriented `key: value` text; nesting by indentation; lists as
  repeated singular keys (`card:` repeated); `#` comments; first line
  `mse version: X.Y.Z` enables backward-compat handling.
- A **reflection system** is the single source of truth per class: one declarative
  member list drives reading, writing, *and* script member access
  (`IMPLEMENT_REFLECTION` + `REFLECT(member)` macros, with `REFLECT_COMPAT` for old
  versions).
- `Scriptable<T>` properties accept either a literal value or a script.
- Sets in directory form write each card to its own file via `include_file:`.

## 5. Reactivity (summary — details in doc 03)

Push-based reactive graph: static dependency extraction at load time (abstract
interpretation of scripts) builds reverse edges on each Field; at runtime the script
manager re-runs dependents transitively, gated by "did the value actually change" and
an age stamp (handles diamonds, converges in one pass).

## Reimplementation takeaways

- **Field/Value/Style separation is the backbone.** A web model should mirror it:
  a Game schema, per-card value records, and a stylesheet style map keyed by stable
  field ids (prefer explicit ids over MSE2's positional index alignment).
- **Reactivity ≈ signals/computed values** — a solved problem on the web; the novel
  part is deriving dependencies from user-authored template scripts.
- **Undo is command-pattern** with mergeable actions and save-point tracking; the
  script manager reacting to the same action stream is an elegant decoupling worth
  keeping.
- **Packages = zip + text + assets.** A web version can use the same shape
  (JSON/YAML + blobs in a zip) with an anti-corruption layer if we ever import real
  MSE2 packages.

Key entry files: `src/data/set.hpp`, `src/data/field.hpp`, `src/util/io/package.hpp`,
`src/util/reflect.hpp`, `src/script/script_manager.cpp`, `src/util/action_stack.hpp`.
