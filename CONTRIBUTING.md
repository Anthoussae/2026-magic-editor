# Contributing & Development Process

This document is the map of how we work on this project. It's written for
James (and anyone else joining) — the repo is deliberately run like a
professional project, as a teaching vehicle. Tools referenced below (justfile,
CI, releases) land with milestone M1; the *process* applies from day one.

## The development loop

Work happens in a plan → implement → review → push cycle, driven by the
repo-local agent skills in [.claude/skills/](.claude/skills/). In Claude Code:

| Step | Command | What it produces |
|---|---|---|
| 1. Plan | `/yona-plan <idea or milestone file>` | A planning directory under `docs/plans/<date>-<name>/` with `notes.md` (discovery, decisions) and `plan.md` (+ phase files) |
| 2. Implement | `/yona-implement <plan path>` | The code, validated; `_DONE.md` completion log; ADRs when warranted; plan archived to `docs/archive/plans/` |
| 3. Review | `/yona-review <branch/PR/diff>` | Review artifacts under `docs/reviews/<date>-<topic>/` with severity-rated findings |
| 4. Push | `/yona-push` | Branch pushed, PR created, CI watched until green |

Rules of the loop:

- **Nothing significant is built without a plan.** Small fixes can skip
  formal planning; anything with design decisions gets a plan first.
- **The current roadmap** is
  [docs/plans/2026-07-17-vertical-slice-roadmap/plan.md](docs/plans/2026-07-17-vertical-slice-roadmap/plan.md)
  (milestones M1–M10). Milestones M6–M8 require their own planning run
  before implementation.
- **Decisions with lasting consequences get an ADR** in
  `docs/adr/YYYY-MM-DD-short-title.md` — what was decided, what the
  alternatives were, why. ADRs are committed with the change that makes the
  decision.
- Agents don't commit unless asked; humans review before merge.

## Where things live

| Path | Contents |
|---|---|
| `docs/plans/` | Active planning directories |
| `docs/archive/plans/` | Completed/superseded plans (with `_DONE.md` logs) |
| `docs/adr/` | Architecture Decision Records |
| `docs/reviews/` | Review artifacts (archived to `docs/archive/reviews/`) |
| `docs/dev/` | How-we-work guides (SDLC, stories, rendering — arrive with their milestones) |
| `docs/mse2/` | Analysis of the original MSE2 codebase |
| `crates/` | Engine crates (`mse-*`) — pure Rust, host-testable |
| `app-web/` | The Dioxus web app (the only wasm-bound crate) |

## Branches, commits, PRs, releases

- All work goes through **PRs into `main`** — never push directly to `main`.
- **Commits**: imperative subject ≤72 chars; body explains *why*; reference
  the milestone when applicable (e.g. `[M4]`); agent-authored commits end
  with `Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>`.
- **CI** (from M1): `pre-merge` runs `just ci` (fmt, clippy with warnings
  denied, tests, wasm build) on every PR and must be green to merge.
- **Releases are automatic** (from M1): merging to `main` tags the next
  CalVer version (`vYYYY.MM.DD-N`), creates a GitHub Release with generated
  notes, and deploys GitHub Pages. Never create tags or releases by hand.

## Validation

`just ci` is the baseline gate — run it before calling any work done. Beyond
that: engine crates carry conformance tests against MSE2's own spec/test
material; rendering uses golden-image tests; UI components ship with stories.
Don't weaken lints, skip tests, or suppress warnings to get green.

## Licensing rules (important — read before writing code)

This project mixes two licenses on purpose (see the licensing ADR, M1):

- **Ported code** (derived from [Magic Set Editor 2](https://github.com/twanvl/MagicSetEditor2),
  GPL-2-or-later): lives in the engine crates, licensed **GPL-3.0-or-later**.
  Every ported file starts with
  `// SPDX-License-Identifier: GPL-3.0-or-later` and
  `// Ported from MagicSetEditor2 <original path>`.
- **Original code** (everything we write fresh): **AGPL-3.0-only**, header
  `// SPDX-License-Identifier: AGPL-3.0-only`.

Don't blend the two in one file. When porting from a new area of MSE2, check
that `NOTICE` still describes the derivation accurately.

## Asset & IP policy (strict)

We keep our code and third-party content fully separate — in the repo, in CI,
at runtime, and in deployment:

- **Never** commit, bundle, fetch, hot-link, or host third-party template
  assets (community templates, WotC artwork, fonts we don't have license to
  redistribute). The app loads such content only from **user-provided files**.
- Our own original demo template assets are the only assets in the repo, each
  accounted for in `assets/ATTRIBUTION.md`.
- Real community templates used for local compatibility testing go in
  `fixtures/local/` (gitignored); tests skip cleanly when they're absent.
- Docs may link to community download sources; code may not.

## UI work is story-driven (from M2)

Every UI component is developed and reviewed through stories (isolated
renderings of its states — empty, loaded, error, long content) before being
composed into the app. A component without stories is incomplete, the same
way code without tests is. See `docs/dev/stories.md` (arrives with M2).

## Teaching norm

When your change introduces a new concept, tool, or workflow, update or add
the matching `docs/dev/` page. The repo should always be able to explain
itself to a newcomer — that's a feature, not overhead.
