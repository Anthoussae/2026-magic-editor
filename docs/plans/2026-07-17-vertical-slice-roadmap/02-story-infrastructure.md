# M2 — Story-Driven UI Infrastructure

- **ID**: M2 · **Size**: md · **Review gate**: end-of-milestone
- **Dependencies**: M1 (workspace, CI, deploy).
- **ADR expectation**: expected (ADR: story infrastructure shape for Dioxus).

## Scope

Bring the storybook concept to this repo so every UI component from M3 onward
is developed story-first. Reference implementation: lp2025's
`lp-app/lpa-studio-web` (`src/stories/`) + `lpa-studio-web-story-macros`
(Yona authorizes reuse under any license; our copy is AGPL-3.0).

Out of scope: real app components (M3+ provide those), visual regression
tooling (future work).

## Teaching goal (primary driver — Yona's explicit priority)

Teach James story-driven frontend development: build components in isolation,
enumerate their states as stories, review visually, then compose into the app.
Deliverable: `docs/dev/stories.md` explaining the workflow, plus a
walk-through-able example story.

## Work items

1. **Evaluate and decide shape** (read lp2025's story host + macros first):
   copy/adapt the lp2025 story crates vs. a fresh minimal story host. Record
   in ADR. Criteria: minimal surface for a new coder, Dioxus 0.7.9 compat,
   no heavy deps in engine crates.
2. **Story host**: a story browser in the web app — routes or a `/stories`
   section listing registered stories, rendering one at a time with
   name/description/knobs (start minimal: name + render; knobs later).
3. **Registration ergonomics**: macro or simple registry so a story is ~10
   lines. Stories live next to their components (`stories.rs` or
   `component/stories/`).
4. **Example stories**: 2–3 for trivial components (e.g. version badge, WIP
   banner from M1) proving the loop.
5. **CI/deploy integration**: stories build in CI; deployed Pages site exposes
   the story browser (e.g. `/stories` route) so reviews can happen on the
   public URL.
6. **Docs**: `docs/dev/stories.md` (what stories are, why, how to add one,
   how review works in the SDLC).

## Validation

`just ci` green; stories visible locally (`just serve-web`) and on the
deployed Pages site; adding a new story takes ≤10 lines (demonstrated).

## Review gate (end of milestone)

Yona reviews the story-host shape + ADR (it sets the frontend workflow for
the whole project) and walks James through adding a story.

## Agent reminders

Do not commit unless asked. Do not expand scope (no knobs/controls gold-plating).
Stop and report if lp2025 code doesn't adapt cleanly. Report changes,
validation, deviations.

## Definition of done

Story browser deployed; example stories exist; ADR + docs written; review
gate passed.
