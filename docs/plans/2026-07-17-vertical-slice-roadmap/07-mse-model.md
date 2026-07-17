# M7 — `mse-model`: Domain Model + Reactive Script Manager

- **ID**: M7 · **Size**: **lg — run `yona-plan` before implementing.** This
  file is a roadmap stub defining scope and exit criteria for that planning
  run.
- **Dependencies**: M4, M6. (M5's demo template is the friendly test target.)
- **ADR expectation**: expected (ownership/identity model — how Rust replaces
  MSE2's intrusive-pointer object graph; likely arenas/ids + change events).

## Scope for the future planning run

Port MSE2's domain layer to `crates/mse-model` (GPL-3.0-or-later):

- Entities: Set, Card, Game, StyleSheet + package loading on top of
  `mse-format` (full schemas via typed deserialization of the node tree).
- Field/Value/Style triad with all slice-relevant field types (text, choice,
  multiple choice, boolean, color, image, symbol, package choice, info).
  Prefer stable field ids over MSE2's positional index alignment; keep an
  index-compat shim where formats require it.
- `Defaultable` values, tagged-string handling (`src/util/tagged_string.cpp`).
- Script manager: init scripts, context variable binding (`card`, `set`,
  `game`, `stylesheet`, `value`), dependency-graph construction (via M6's
  analysis), age-stamped change-gated propagation
  (`src/script/script_manager.cpp`).
- Action/undo core (`src/util/action_stack.hpp` + value actions): port the
  *model-level* command pattern now (cheap while porting, needed for editing
  later); UI exposure stays out of scope.

Out of scope: keywords expansion, statistics, packs, export (post-slice) —
but leave typed stubs where Game schema references them so real templates
still *parse*.

## Exit criteria

1. Load the M5 demo game+style+set end-to-end: all values present, computed
   fields evaluated, dependency updates propagate on programmatic edits.
2. Load a real community template + set from `fixtures/local/` to the same
   standard for an agreed field subset (compat test, skip-if-absent).
3. Model API sufficient for M8 rendering (styles resolvable per card) and M9
   listing (card list with sort field).

## Agent reminders

Do not implement from this stub. Run `yona-plan` first. Reference analysis:
`docs/mse2/02-domain-model.md`; C++: `src/data/**`.
