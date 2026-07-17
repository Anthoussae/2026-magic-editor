# Notes — Vertical Slice Roadmap (2026-07-17)

## Initial understanding and goals

Build a web-based card editor by porting the MSE2 engine to Rust. This roadmap
targets the first **vertical slice**: load real MSE2 template packages and a card
set, render cards in the browser, deployed to GitHub Pages.

Decisions already made in conversation (pre-planning):

- **Port, not reimagine**: port the MSE2 engine (C++ at
  `/Users/yona/dev/james/MagicSetEditor2`) semi-mechanically to Rust. The engine
  is `src/data` + `src/script` + `src/util` (~33k LOC C++). The wxWidgets GUI
  (~24.5k LOC) is a rewrite in Dioxus regardless; rendering (~7k LOC) is an
  algorithmic port onto a new graphics substrate.
- **Compatibility with existing MSE2 assets is the north star** — the community
  template ecosystem is the moat.
- **Stack**: Rust workspace, Dioxus web UI, tiny-skia + cosmic-text rendering
  (host-testable, deterministic export), OPFS browser storage later.
- **Licensing**: MSE2 source headers grant GPL-2-or-later (verified, 164 files).
  Ported engine crates → GPL-3.0-or-later; original code → AGPL-3.0. Compatible
  via GPLv3 §13 / AGPLv3 §13.
- **Audience**: the project is for James, who is new to coding; Yona advises.
  Roadmap increments must be executable/followable by a newer coder using the
  repo-local yona-* skills.
- Yona (author of lp2025 at `/Users/yona/dev/photomancer/lp2025`) authorizes
  reuse of any lp2025 code under whatever license is needed.

## Current state

- Repo: analysis docs only (`docs/mse2/01..04`), no code. Git + GitHub
  (`Yona-Appletree/2026-magic-editor`, public). No LICENSE files yet.
- yona-* skills copied to `.claude/skills/`.
- MSE2 source checkout available locally for reference; its `data/` contains
  only `en.mse-locale` — **no game/style template packages in the source repo**.
- MSE2 has a test suite usable as an oracle: `test/script/*.mse-script` plus
  format specs in `doc/type/*.txt`, `doc/script/`, `doc/function/`.

## Relevant references inspected

- `docs/mse2/*.md` — full analysis (domain model, script system, rendering, UI).
- lp2025: Rust workspace with Dioxus web app (`lp-app/lpa-studio-web` with
  `stories/` + `lpa-studio-web-story-macros` — story-driven Dioxus components),
  `lpa-fs-opfs` (OPFS storage), host-testable crate philosophy (heavy deps like
  wgpu kept out of default-members).

## Discovery findings

- **Templates are not in MSE2 releases**: GitHub releases contain executables
  only; 2.1.0 notes say "No templates are included." Templates are downloaded
  from https://magicseteditor.boards.net/board/5/additional-downloads (fan
  forum). ⇒ obtaining fixtures is a manual step; redistribution rights are
  unclear (community content + WotC IP in magic frames/mana symbols).
- **Consequence for the deployed slice**: the public GitHub Pages demo cannot
  legally ship WotC assets, so the deployed demo must use an original template
  we author; real magic templates remain local-only dev fixtures.
- lp2025 uses **Dioxus 0.7.9**, a `justfile` task runner, `rust-toolchain.toml`,
  and a strong AGENTS.md convention — all good patterns to mirror minimally.
- lp2025's story system (`lpa-studio-web` `stories/` + story-macros crate) is
  reusable, but the slice has almost no interactive UI; story infra can wait.

## Open questions (to triage with user)

- Q-fixtures: where to obtain real template packages (magic.mse-game, a
  stylesheet, symbol fonts) and whether they can live in the public repo
  (WotC IP concerns for mana symbols/frames) vs. gitignored local fixtures +
  an original test template for CI.
- Q-scope: vertical slice is read-only (viewer, no editing) — confirm.
- Q-conventions: adopt lp2025 conventions (justfile, AGENTS.md, story macros)
  directly, copy code, or start minimal?
- Q-deploy: GitHub Pages via Actions with dioxus-cli; confirm.
- Q-crate-naming: `mse-*` engine crates + app crate naming.
- Q-script-fidelity: port order for script VM and how much of the builtin
  function library the slice needs.

## Answers / scope changes

**2026-07-17, from Yona:**

- Q1 read-only slice: **yes**.
- Q2 crate naming (`mse-format`/`mse-script`/`mse-model`/`mse-render` + `app-web`): **yes**.
- Q3 mirror lp2025 conventions minimally, written fresh: **yes**.
- Q4 GitHub Actions → Pages via dioxus-cli: **yes**.
- Q5 licensing files in first milestone: **yes**.
- Q6 MSE2 script test suite as VM conformance oracle: **yes**.
- Q7 defer story tooling: **NO — overridden.** Story infrastructure comes
  *early*, as its own work item (bring the storybook concept over from lp2025).
  Rationale: story-driven development is a core part of the frontend SDLC Yona
  wants to teach James.
- **New scope — README**: needs a WIP statement, very clear attribution
  (modeled on lp2025's README), and a clear statement about the use of agentic
  tools in building the project.
- Q8 fixture/IP strategy: **agreed, and strengthened.** Code and third-party
  assets must be fully separate *throughout, including at runtime*: the app or
  the user downloads assets; they are never bundled with our code. If
  cross-domain access is a problem, separately-hosted assets would have to be
  under a totally different legal entity, not associated with Yona's name or
  the project (escape hatch, not the plan).
- Asset acquisition model ("emulator ROM problem"): resolved — see below.
  Docs may *link* to the community forum; the app itself uses a
  **user-provided-file model** (drag-drop/file-picker, stored in OPFS).
  No auto-fetch script in the app or repo for third-party templates.

**2026-07-17, milestone review — Yona: lgtm, plus additions:**

- **Skills customization pass** (requested, done during planning session):
  customize the imported `.claude/skills/yona-*` files for this repo — commit
  language, plan archiving, ADR location, validation commands.
- **M1 expanded — professional SDLC from lp2025**: bring over the LightPlayer
  SDLC: CalVer versioning (`vYYYY.MM.DD-N` tags auto-created on main push via
  `scripts/tag-next-version.sh` pattern), `print-app-version.sh` pattern,
  automatic GitHub Releases with generated notes, pre-merge CI workflow +
  main-push workflow, Pages deploy. Explicit teaching goal: show James a
  professional release pipeline.
- lp2025 reference workflows: `.github/workflows/{pre-merge,main-push,deploy-studio-pages}.yml`,
  `scripts/{tag-next-version,print-app-version}.sh`.

**2026-07-17, after plan written — Yona:** wants an explicit
SDLC/CONTRIBUTING doc making the process clear for James and others.
→ Root `CONTRIBUTING.md` written during planning (process map: yona-* loop,
repo layout, commit/PR/release conventions, licensing rules, asset policy,
story-driven UI, teaching norm). M1 adjusted to *update* it alongside
`docs/dev/sdlc.md` rather than create it.

## Assumptions made without explicit confirmation

- Commit message convention (skills pass): imperative subject ≤72 chars, body
  explains *why*, reference plan/milestone (e.g. `[M1]`) when applicable,
  `Co-Authored-By: Claude` trailer on agent-authored commits. Adjust if wanted.
- ADRs live in `docs/adr/YYYY-MM-DD-short-title.md` (yona-implement's
  convention; MADR-lite format).
- Completed plans archive to `docs/archive/plans/` (per yona-plan skill default).

## Future work ideas

- Editing UI (WYSIWYG per-field editors), undo stack surfaced in UI.
- Keywords panel, statistics, pack generator, symbol editor.
- MSE2 `.mse-set` zip import/export in browser (drag & drop).
- OPFS-backed set library; sharing via URL.
- Backend (sync/share) — motivates AGPL.
