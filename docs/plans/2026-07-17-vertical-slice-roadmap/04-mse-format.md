# M4 — `mse-format`: Package Model + Format Reader

- **ID**: M4 · **Size**: md · **Review gate**: none
- **Dependencies**: M1. Independent of M2/M3 (parallelizable).
- **ADR expectation**: possible (only if reader design diverges from MSE2's
  reflection approach in ways that constrain later crates).

## Scope

First ported crate: `crates/mse-format` (GPL-3.0-or-later, SPDX + provenance
headers per M1 conventions). Reads MSE2's tab-indented text format and
package containers (directory or zip).

Out of scope: script parsing (values that are scripts are captured as raw
strings for M6), the domain model itself (M7), writing/saving beyond
round-trip tests.

## Reference material (read before implementing)

- C++: `src/util/io/reader.cpp`, `writer.cpp`, `package.cpp/.hpp`,
  `reflect.hpp` in the MSE2 checkout.
- Spec: `doc/file/format.txt`, `doc/file/package.txt`; per-type schemas in
  `doc/type/*.txt`.
- Analysis: `docs/mse2/02-domain-model.md` §4.
- Live example: MSE2's `data/en.mse-locale/locale`.

## Work items

1. **Text format reader**: tab-indented `key: value` blocks → a generic
   node/tree representation. Handle: canonical key normalization (case,
   spaces↔underscores), `#` comments, blank lines, repeated singular keys as
   lists, multiline values (indented continuation), `mse version:` header
   capture, 8-spaces-as-tab tolerance (with warning), `include_file:`
   indirection (resolved via the package).
2. **Writer** (for round-trip tests): tree → text, byte-stable for files we
   read (lossless round-trip where MSE2 is lossless).
3. **Package container**: open directory or zip (`zip` crate) transparently;
   list/read files; main data file by package type name (`set`, `game`,
   `stylesheet`, `locale`, ...); `LocalFileName`-style asset references
   resolved to bytes. Pure API — no filesystem assumptions that break wasm
   (byte-source trait; dir/zip/in-memory backends).
4. **Typed layer (thin)**: serde-like mapping from the node tree onto plain
   structs for *package headers only* (name, version, dependencies, icon).
   Full schemas belong to M7.
5. **Tests**: parse `en.mse-locale/locale` (copy small excerpts as committed
   fixtures — MSE2 source is GPL, fine to include with provenance note);
   round-trip tests; error/warning cases from `doc/file/format.txt`.

## Conventions

Engine crate rules (AGENTS.md): no wasm/app deps, host-native tests, GPL
SPDX headers with `Ported from MagicSetEditor2 src/util/io/...` notes.

## Validation

`just ci` green; round-trip property holds on all fixtures; crate README
documents the format with examples.

## Agent reminders

Do not commit unless asked. Do not widen into schema/domain modeling. Match
MSE2 quirks exactly (they are compat requirements, not bugs to fix) — record
each discovered quirk in the crate README. Stop and report on spec/source
contradictions.

## Definition of done

Reader/writer/package APIs with tests; locale fixture parses; docs written;
CI green.
