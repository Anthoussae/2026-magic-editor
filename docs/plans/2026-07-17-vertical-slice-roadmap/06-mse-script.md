# M6 — `mse-script`: Parser, VM, Builtins, Conformance

- **ID**: M6 · **Size**: **lg — run `yona-plan` before implementing.** This
  file is a roadmap stub defining scope and exit criteria for that planning
  run, not an implementation guide.
- **Dependencies**: M4 (raw script strings arrive via `mse-format`).
  Parallelizable with M5.
- **ADR expectation**: possible (value semantics, error model, dependency
  analysis design).

## Scope for the future planning run

Port MSE2's scripting engine to `crates/mse-script` (GPL-3.0-or-later):

- Tokenizer + parser (including string-mode `{...}` interpolation and
  `include file:` handling) — C++ `src/script/parser.cpp`.
- Bytecode + stack VM with dynamic scoping, closures/default args, proper
  tail calls — `src/script/script.hpp`, `context.cpp`.
- Value system: nil/int/double/bool/string/color/list/map/function/regex/
  date/image handles, with MSE2's exact coercion + `nil`-identity semantics —
  `src/script/value.cpp`.
- Builtins **needed by template loading/rendering** (basic, regex via
  `fancy-regex` for backreferences, editor-choice helpers, image-function
  *graph construction* — actual rasterization lands in M8). Explicitly
  deferred: export, spelling, english-NLP builtins.
- Dependency analysis (abstract evaluation) — `src/script/dependency.cpp` —
  API consumed by M7's script manager.
- **Conformance suite**: port `test/script/*.mse-script` from the MSE2 repo
  as the oracle; every VM feature lands with its conformance tests.

## Exit criteria (for the whole milestone)

1. Conformance suite passes.
2. Scripts extracted from a real community template (local fixture) parse and
   evaluate for a representative sample agreed during the planning run.
3. Dependency-analysis API documented and consumed by a prototype in M7.

## Notes for the planning run

- Reference analysis: `docs/mse2/03-scripting-and-templates.md`.
- Spec: `doc/script/`, `doc/function/` in the MSE2 checkout.
- Biggest fidelity risks to plan around: dynamic scoping interactions,
  regex flavor differences, string-mode edge cases, image-function laziness.
- Sub-phase naturally: values+parser → VM core → builtins (grouped) →
  dependency analysis → fixture-driven gap-filling.

## Agent reminders

Do not implement from this stub. Run `yona-plan` scoped to this milestone
first; it should produce its own planning directory referencing this file.
