# MSE2 Analysis: Scripting & Template System

> Source: `/Users/yona/dev/james/MagicSetEditor2` — `src/script/`, `doc/script/`, `doc/type/`, `doc/function/`.
> Note: the actual `magic.mse-game` / `magic-new.mse-style` template packages are **not** in the
> source checkout (only `data/en.mse-locale/`); the formats are fully specified in `doc/type/*.txt`.

MSE2 is best understood as a **template engine with a GUI**. Games and card layouts are
not hardcoded — they are data packages containing field definitions, layout styles, and
scripts in a small custom language. The C++ core is a generic engine that loads these.

## 1. The scripting language

A small, expression-oriented dynamic language (documented in `doc/script/`):

- **Values**: `nil`, ints, doubles, booleans, strings, colors (`rgb(...)`), lists `[a,b,c]`,
  maps `[key: value]`, first-class functions `{ ... }`, regexes, dates, images.
  `nil` acts as an identity element (adding it is a no-op).
- **Operators**: arithmetic; `+` overloaded for concat and *function composition*
  (`(f+g)() == g(input: f())`); short-circuit booleans; member access `a.b`;
  assignment `:=`; sequencing `;`.
- **Control flow (all expressions)**: `if/then/else`, `case of`, `for each x in list do ...`
  (loop results combined with `+`).
- **Dynamic scoping**: assignments in a caller are visible in callees. The implicit
  argument is `input`; named arguments are supported; `f@(arg: default)` creates a
  partially-applied closure.
- **String mode**: strings can embed script with `{ ... }` — e.g.
  `"uppercase of {input} is {to_upper()}"`. This is the basis of "scriptable" template
  properties.

### Implementation (relevant if we ever want compatibility)

Not a tree-walker: scripts are compiled to **bytecode run on a stack VM**.

- Parser: `src/script/parser.cpp` (hand-written tokenizer + recursive descent, emits instructions;
  handles `include file:` directives at parse time).
- Bytecode: `src/script/script.hpp` — 6-bit opcode + 26-bit operand instructions
  (`I_PUSH_CONST`, `I_JUMP*`, `I_GET_VAR/I_SET_VAR`, `I_CALL/I_CLOSURE/I_TAILCALL` with proper
  tail calls, `I_LOOP`, etc.).
- VM: `src/script/context.cpp` — value stack, integer-indexed variables, scope stack for
  dynamic scoping.
- Value system: `src/script/value.hpp` — `ScriptValue` base class with virtual coercions
  (`toString/toInt/toImage/...`), member access, and iteration.

## 2. Built-in function library (`src/script/functions/`)

- **basic.cpp** — conversions (`to_string`, `to_int`, ...), case/title (`to_upper`, `to_title`,
  `curly_quotes`), string ops (`substring`, `contains`, `format`, ...), list ops (`filter_list`,
  `sort_list`), tag handling (`tag_contents`, `remove_tags`), keywords (`expand_keywords`),
  math, randomness, diagnostics (`warning`, `assert`).
- **regex.cpp** — `match_text`, `replace_text`, `filter_text`, `split_text` (+ reusable
  `*_rule` closure variants).
- **image.cpp** — image generation pipeline returning `GeneratedImage` values:
  blends, masks, crop/rotate/flip, `drop_shadow`, `recolor_image`, `symbol_variation`.
- **editor.cpp** — choice-field helpers (`chosen`, `exclusive_choice`, `require_choice`) and
  `combined_editor` (merges several fields into one text editor).
- **export.cpp** — `to_html`, `to_text`, `to_bbcode`, `write_text_file`, `write_image_file`, ...
- **english.cpp** — English NLP for keyword reminder text (`english_number`, `english_plural`).
- **spelling.cpp** — hunspell-backed spellcheck.

Per-function reference: `doc/function/*.txt`.

## 3. How templates use scripts

Scripts appear wherever a property is declared `script` or `scriptable` in the format docs:

- **Field post-processing**: every field has an optional `script:` applied after each edit
  (`value` holds the content — e.g. auto-capitalization via `to_title` + `replace_text`
  composed with `+`), and a `default:` script computing the value while unedited
  (e.g. `default: set.border_color`).
- **Scriptable layout**: style properties (coordinates, fonts, visibility, masks) can be
  scripts: `left: { if card.name == "" then 100 else 123 }`.
- **Tagged text**: text values are stored as *tagged strings* (`doc/type/tagged_string.txt`)
  with tags like `<b>`, `<i>`, `<sym>`, `<color:...>`, `<atom>`, `<kw-...>`. Mana symbols are
  `<sym>`-tagged text rendered by a **symbol font** package (longest-match splitting of
  e.g. `W/G` before `W`).
- **Keywords**: games define keywords with a `match:` pattern containing typed parameter
  atoms (`<atom-param>cost</atom-param>`) and a `reminder:` script using captured params
  (`{param1}`). `expand_keywords` inserts reminder text into rules text automatically.

## 4. Reactive dependency tracking (the standout mechanism)

MSE2 implements an **incremental dataflow graph** between scripted values
(`src/script/dependency.*`, `src/script/script_manager.cpp`, `src/data/field.cpp`):

1. **Static analysis at load time**: every script is executed once in an abstract
   "dependency mode" (dummy values, both branches of conditionals taken). When the abstract
   run touches `card.name`, the `name` field records a reverse edge in its
   `dependent_scripts` list ("who must re-run if I change").
2. **Runtime propagation**: on any edit, the script manager re-runs the edited value's
   script, then walks the reverse edges, re-running dependents — and only if a value
   *actually changed* does it enqueue that value's own dependents (transitive cascade with
   an age stamp preventing double updates per round). Changed values fire events so the
   UI/renderer refreshes.

Dependencies are typed (`DEP_CARD_FIELD`, `DEP_SET_FIELD`, `DEP_CARD_STYLE`, ...) so a
change can target one card's value, a set-wide value, all cards, or invalidate a style /
generated image.

**Web-app takeaway**: this maps almost 1:1 onto modern reactive/signal systems
(computed values + fine-grained invalidation). We get the equivalent mechanism nearly for
free from a signals library or a spreadsheet-style computed graph — but note MSE2 derives
the graph by *abstract interpretation of user scripts*, which is what lets template authors
write plain expressions without declaring dependencies.

## 5. Package types (the template ecosystem)

All are directories/zips containing one reflected data file in MSE's indentation-based
text format; the extension selects the schema:

| Package | Contains |
|---|---|
| `.mse-game` | The *rules-side description*: set fields, card fields, init script, keywords, keyword parameter types, word lists, statistics dimensions, pack types |
| `.mse-style` | The *look*: target game, card dimensions/dpi, one `style` block per card field (position, font, mask, z-index — mostly scriptable), styling options |
| `.mse-symbol-font` | Image-based font for `<sym>` text (mana symbols): symbol images + splitting rules + insert-symbol menu |
| `.mse-export-template` | A script producing HTML/text export, plus user-visible option fields |
| `.mse-locale` | UI translation maps (menus, tooltips, errors, per-package field name translations) |
| `.mse-include` | Shared scripts/images pulled in via `include file:` |
| `.mse-set` | A user's set: chosen game + stylesheet + card values + embedded images |

Packages declare versioned dependencies on each other (`depends on: magic.mse-game 2007-06-06`).

### Example: field definition (`doc/type/field.txt`)

```
card field:
	type: color
	name: border color
	default: set.border_color
	choice: { name: black, color: rgb(0,0,0) }
	show statistics: false
```

### Example: keyword (`doc/type/keyword.txt`)

```
keyword:
	keyword: Equip
	match: Equip <atom-param>cost</atom-param>
	mode: core
	reminder: {param1}: Attach to target creature you control. Equip only as a sorcery.
```
