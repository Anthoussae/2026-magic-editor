# M1 — Repo Foundation

- **ID**: M1 · **Size**: md · **Review gate**: end-of-milestone (see below)
- **Dependencies**: none. Everything else depends on this.
- **ADR expectation**: expected (two: licensing split; CalVer/release pipeline).

## Scope

Turn the docs-only repo into a professional Rust project skeleton with the
full SDLC in place — licensing, README, workspace, CI/CD, CalVer releases,
Pages deploy of a placeholder app. **No engine code.**

Out of scope: any porting, any real UI, story infrastructure (M2).

## Teaching goal

James sees what a professionally set-up repo looks like *before* feature work:
licensing hygiene, CI gates, versioned releases with notes, deploy pipeline.
Write `docs/dev/sdlc.md` explaining the pipeline as part of this milestone.

## Work items

1. **Licensing**
   - Root `LICENSE`: AGPL-3.0-only text. `licenses/GPL-3.0-or-later.txt` for
     engine crates (each engine crate gets its own `LICENSE` copy + README
     licensing note when created).
   - `NOTICE`: attribution to Magic Set Editor 2 authors (Twan van Laarhoven
     et al., GPL-2-or-later, https://github.com/twanvl/MagicSetEditor2),
     statement that engine crates are derivative works distributed under
     GPL-3.0-or-later; lp2025 attribution (author-authorized reuse).
   - SPDX header convention (already documented in `CONTRIBUTING.md`, which
     exists — update it if details change here):
     - ported code: `// SPDX-License-Identifier: GPL-3.0-or-later` + `// Ported from MagicSetEditor2 <path>`
     - original code: `// SPDX-License-Identifier: AGPL-3.0-only`
   - Licensing ADR (`docs/adr/YYYY-MM-DD-licensing-split.md`): rationale for
     AGPL app / GPL engine, §13 combination.
2. **README rewrite** (root). Must include, modeled on lp2025's README:
   - Prominent **WIP statement** (early development, not usable yet).
   - What it is: web-based card editor, Rust port of the MSE2 engine.
   - **Attribution section**: clear credit to MSE2 and authors, link, license
     lineage (GPL-2-or-later → GPL-3.0-or-later engine crates; AGPL-3.0 app).
   - **Agentic tools statement**: this project is built substantially with AI
     coding agents (Claude Code) under human direction, as a deliberate
     teaching/workflow choice; link `docs/dev/` workflow docs.
   - **Asset policy**: no third-party template assets in repo/deploy; the app
     loads user-provided files only.
   - Quickstart (`just` recipes), docs index link, license section.
3. **Rust workspace**
   - `Cargo.toml` workspace: members `app-web` (placeholder Dioxus 0.7 app)
     only, `crates/` dir reserved for engine crates (created in M3/M4).
   - `rust-toolchain.toml` (pinned stable), `rustfmt.toml`, workspace lints
     (`[workspace.lints]`, clippy warnings deny), `.gitignore` update
     (`target/`, `dist/`, `fixtures/local/`).
   - Placeholder `app-web`: Dioxus 0.7 hello page showing app name + version
     (from build-time env, see item 5) + WIP notice. Builds with `dx build`.
4. **justfile + AGENTS.md**
   - `just` recipes: `check` (fmt+clippy), `test`, `build-web`, `serve-web`,
     `ci` (what CI runs), `version` (print current version).
   - `AGENTS.md`: project intent, crate map (planned), conventions (SPDX
     headers, commit style, yona-* skill workflow, asset/IP policy,
     "engine crates never depend on app/wasm").
5. **CalVer + CI/CD** (port the lp2025 pattern; reference
   `/Users/yona/dev/photomancer/lp2025/scripts/tag-next-version.sh`,
   `print-app-version.sh`, `.github/workflows/{pre-merge,main-push}.yml`)
   - `scripts/tag-next-version.sh`: tag `vYYYY.MM.DD-N` on main.
   - `scripts/print-app-version.sh`: tag → version, else `branch@sha`.
   - `.github/workflows/pre-merge.yml`: PRs → `just ci` (fmt, clippy, test,
     wasm build).
   - `.github/workflows/main-push.yml`: main → tag next version → GitHub
     Release with auto-generated notes (`gh release create --generate-notes`)
     → build → deploy Pages (`actions/deploy-pages`, project-site base path
     `/2026-magic-editor/`).
   - App displays its version (env injected at build from
     `print-app-version.sh`).
   - Release-pipeline ADR (`docs/adr/YYYY-MM-DD-calver-release-pipeline.md`).
   - `docs/dev/sdlc.md`: how versioning/CI/releases/deploy work here, written
     for a newer coder. Cross-check `CONTRIBUTING.md` (exists) still matches
     the implemented pipeline; update where reality differs.
6. **Repo settings** (needs Yona or `gh` with admin): enable Pages
   (GitHub Actions source), branch protection on main requiring pre-merge
   checks. Document in `docs/dev/sdlc.md`; flag anything the agent cannot do.

## Conventions for this milestone

- Commit style: imperative subject ≤72 chars; body says why; reference
  milestone (`[M1]`); `Co-Authored-By: Claude <...>` trailer on agent-authored
  commits.
- Keep the placeholder app minimal — resist adding UI polish here.

## Validation

- `just ci` passes locally (fmt, clippy, tests, `dx build`).
- Push to a branch → pre-merge workflow green.
- Merge to main → tag created, Release with notes exists, Pages site live and
  showing version.
- `grep -r "SPDX" app-web/src` shows headers; LICENSE/NOTICE/README present.

## Review gate (end of milestone)

Stop and review with Yona (+ James): README wording (WIP/attribution/agentic
statements), both ADRs, the live Pages URL, and repo settings that needed
manual action. These set public-facing tone and are cheap to fix now.

## Agent reminders

Do not commit unless asked. Do not expand scope. Do not suppress warnings or
disable tests. Stop and report if blocked (e.g. missing repo permissions).
Report what changed, what was validated, and deviations.

## Definition of done

All work items done, validation green, review gate passed, both ADRs merged,
`_DONE.md` appendix per yona-implement.
