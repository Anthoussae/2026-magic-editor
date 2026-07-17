# Documentation

## MSE2 analysis (`mse2/`)

Study of the original Magic Set Editor 2 C++ codebase
(`/Users/yona/dev/james/MagicSetEditor2`) that this project reimagines for the web.

1. [Overview](mse2/01-overview.md) — what MSE2 is, source layout, package
   ecosystem, feature inventory
2. [Domain model](mse2/02-domain-model.md) — Set/Card/Game/StyleSheet, the
   Field/Value/Style triad, undo system, file format
3. [Scripting & templates](mse2/03-scripting-and-templates.md) — the script
   language, how templates use it, the reactive dependency graph
4. [Rendering & UI](mse2/04-rendering-and-ui.md) — card draw pipeline, rich text
   engine, symbol editor, UI structure, export

MSE2 also ships its own excellent technical reference in its repo under `doc/`
(`doc/type/*.txt` for every data type, `doc/script/` for the language,
`doc/function/` for built-ins) — treat that as the authoritative format spec.

## Design (`design/`)

Design documents for this project will live in `docs/design/` as decisions are
made (architecture, domain model, framework choices, vertical-slice scope).
