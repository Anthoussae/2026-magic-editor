---
kind: plan
size: lg
depth: roadmap
status: active
repo: 2026-magic-editor
created: 2026-07-17
adr: expected
---

# Vertical Slice Roadmap — Web Card Editor (MSE2 Rust Port)

## Why this size

`lg` / roadmap. This plan sequences ~10 milestones toward the first vertical
slice. Milestones M6–M8 (script VM, domain model, full rendering) are large
enough to require their own `yona-plan` runs when reached; the others are
executable from their milestone files directly.

## Goal

A deployed, public, working vertical slice: a browser app (GitHub Pages) that
**loads MSE2-format template packages and card sets and displays rendered
cards**, built on a Rust port of the MSE2 engine.

### Acceptance criteria

1. Public GitHub Pages site renders the cards of a bundled, original demo set —
   correct layout, images, styled text, symbol-font symbols.
2. A user can drag-drop (or file-pick) their own MSE2 `.mse-set` + template
   packages; cards render in the browser. No third-party assets are bundled,
   fetched, or hosted by us — user-provided-file model only.
3. Local (non-CI) compatibility tests render cards from real community
   templates (gitignored fixtures) and compare against MSE2-exported golden
   images.
4. Engine crates are host-testable (no wasm needed for tests); CI runs fmt,
   clippy, tests; main-push CI tags CalVer versions, creates GitHub Releases
   with notes, and deploys Pages.
5. Licensing is correct throughout: ported engine crates GPL-3.0-or-later with
   MSE2 attribution; original code AGPL-3.0; NOTICE file present.

## Scope

- Read-only viewing (list + render). Set/card **editing is out of scope**.
- MSE2 format reading. Writing/saving sets is out of scope (beyond what tests
  need).
- Script VM coverage: what real templates need to *load and render*. Exhaustive
  builtin coverage (spellcheck, export functions, english NLP) is out of scope.
- Out of scope for the slice: keywords panel, statistics, pack generator,
  symbol editor, printing, HTML export, MSE1/Apprentice/MWS formats,
  localization UI.

## Background

- Approach and rationale: decided in pre-planning conversation; see
  [notes.md](notes.md).
- MSE2 analysis: [docs/mse2/](../../mse2/01-overview.md) (01–04).
- MSE2 source (reference for the port): a local clone of
  https://github.com/twanvl/MagicSetEditor2 (needed from M4 onward) —
  headers grant GPL-2-or-later; format spec in `doc/type/*.txt`; script
  conformance suite in `test/script/`.
- lp2025: Yona's private project, reference for Dioxus 0.7 app
  structure, story-driven components, OPFS storage, justfile/AGENTS.md
  conventions, CalVer + release CI (`scripts/tag-next-version.sh`,
  `.github/workflows/{pre-merge,main-push,deploy-studio-pages}.yml`). Yona
  (author) authorizes reuse of lp2025 code under any license needed. James
  needs a copy from Yona before M1/M2; if unavailable, implement the same
  concepts from the descriptions in this roadmap.

## Audience note (important for every milestone)

This project is a teaching vehicle for James, a new coder. Yona set the
project up and handed it off (2026-07-18); James now owns the project solo —
every review gate in the milestone files is James's responsibility, as is
merging PRs. Every milestone should:

- Be executable by a newer coder driving AI agents with the repo-local
  `.claude/skills/yona-*` workflow (plan → implement → review → push).
- Prefer explicit, well-named, documented structure over cleverness.
- Update teaching-oriented docs (`docs/`) as concepts land (stories, CalVer,
  hexagonal boundaries), so the repo itself explains how it is built.

## Architecture (decided)

Rust workspace; hexagonal: engine crates are pure and host-testable, the app
crate is the only wasm-bound shell.

| Crate | License | Contents |
|---|---|---|
| `mse-format` | GPL-3.0-or-later (ported) | Package model (dir/zip), tab-indented reader/writer, version compat |
| `mse-script` | GPL-3.0-or-later (ported) | Script parser, bytecode VM, builtins, dependency analysis |
| `mse-model` | GPL-3.0-or-later (ported) | Set/Card/Game/StyleSheet, Field/Value/Style, script manager |
| `mse-render` | GPL-3.0-or-later (ported algorithms) | Card renderer on tiny-skia + cosmic-text |
| `app-web` | AGPL-3.0 | Dioxus 0.7 UI, canvas blit, OPFS storage, file import |
| (stories host) | AGPL-3.0 | Story-driven component development (M2 decides shape) |

Asset/IP policy (decided, see notes): our code and third-party template assets
stay fully separate — in the repo, in CI, at runtime, and in deployment. Docs
may link to community template sources; the app only ever loads user-provided
files. No auto-fetch of third-party assets anywhere.

## Milestones

| Milestone | Size | Summary | Own planning? | File |
|---|---|---|---|---|
| M1 | md | Repo foundation: licenses, README, workspace, CI/CD, CalVer + releases, Pages deploy | No | [01-repo-foundation.md](01-repo-foundation.md) |
| M2 | md | Story-driven UI infrastructure (from lp2025 concept) | No | [02-story-infrastructure.md](02-story-infrastructure.md) |
| M3 | md | Walking-skeleton card render (tiny-skia + cosmic-text → canvas) | No | [03-walking-skeleton-render.md](03-walking-skeleton-render.md) |
| M4 | md | `mse-format`: package + tab-indented format reader | No | [04-mse-format.md](04-mse-format.md) |
| M5 | md | Original demo template + local fixtures pipeline | No | [05-demo-template-and-fixtures.md](05-demo-template-and-fixtures.md) |
| M6 | lg | `mse-script`: parser, VM, builtins, conformance | **Yes** | [06-mse-script.md](06-mse-script.md) |
| M7 | lg | `mse-model`: domain model + reactive script manager | **Yes** | [07-mse-model.md](07-mse-model.md) |
| M8 | lg | Full template-driven card rendering + golden tests | **Yes** | [08-full-rendering.md](08-full-rendering.md) |
| M9 | md | Viewer app: lists, display, user file import (OPFS) | No | [09-viewer-app.md](09-viewer-app.md) |
| M10 | sm | Slice hardening: cleanup, docs, validation, deploy | No | [10-slice-hardening.md](10-slice-hardening.md) |

Sequencing: M1 → M2 → M3 prove workflow + plumbing before any porting. M4 → M6
→ M7 → M8 is the port core (M6–M8 each start with their own `yona-plan` run).
M5 can run in parallel with M6+. M9 needs M7+M8; M10 is last.

## Documentation expected to change

- `README.md` — rewritten in M1 (WIP statement, attribution, agentic-tools
  statement, dev quickstart).
- `docs/README.md` — index grows as docs land.
- New: `docs/adr/` (ADRs), `docs/dev/` (teaching/workflow docs), per-crate
  `README.md`s, `CONTRIBUTING.md`.
- `docs/mse2/` — analysis docs updated if porting reveals inaccuracies.

## ADR expectations

ADRs live in `docs/adr/YYYY-MM-DD-short-title.md` (the yona-implement
convention). Expected during the roadmap:

- Licensing split (AGPL app / GPL engine, MSE2 attribution) — M1.
- CalVer + release pipeline — M1.
- Story infrastructure shape for Dioxus — M2.
- Rendering substrate (tiny-skia + cosmic-text, canvas blit) — M3 (confirming
  the pre-planning decision after the spike proves it).
- Asset/IP separation policy (user-provided-file model) — M5.
- Script VM value semantics / dependency tracking approach — M6/M7 as needed.

## Validation strategy

- Every crate: `cargo test` host-native; `cargo fmt --check`, `cargo clippy`
  (workspace-wide, warnings deny in CI). Wired as `just` recipes in M1.
- `mse-format`: round-trip + spec-conformance tests against `doc/file/` +
  `doc/type/` and `en.mse-locale`.
- `mse-script`: MSE2's `test/script/*.mse-script` suite ported as the
  conformance oracle.
- `mse-render`: golden-image tests (committed goldens for the demo template;
  gitignored local goldens exported from real MSE2 for compat).
- App: wasm build in CI; Pages deploy from main; stories serve as visual
  review surface.
- Final: acceptance criteria walkthrough in M10.

## Agent reminders

- Work on a branch; commit, push, open the PR, and drive CI to green
  autonomously (see `CONTRIBUTING.md`). Humans review and merge the PR.
  Do not expand scope.
- Milestones M6–M8: run `yona-plan` first; do not start implementing from the
  milestone file alone.
- Every ported file needs the GPL-3.0-or-later SPDX header + MSE2 provenance
  note (established in M1).
- Completion is recorded by `yona-implement` in `_DONE.md` here; milestone
  files get an `Implementation Result` appendix when done.
