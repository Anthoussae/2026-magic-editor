# M10 — Slice Hardening: Cleanup, Docs, Validation

- **ID**: M10 · **Size**: sm · **Review gate**: final acceptance review
- **Dependencies**: all prior milestones.
- **ADR expectation**: none (unless cleanup surfaces a decision).

## Scope

The standard final phase: sweep, document, validate, ship.

## Work items

1. **Cleanup sweep** across the workspace:
   - `rg -n "TODO|FIXME|XXX|HACK|dbg!|println!|unwrap\(\)"` — triage each hit
     (fix, ticket into notes.md future work, or justify in place).
   - Commented-out code, scratch files, dead `#[allow(...)]`s, disabled or
     `#[ignore]`d tests, suppressed warnings — remove or justify.
   - Scope creep check against plan.md; licensing header audit
     (`rg -L "SPDX" --files-without-match` pattern per crate).
2. **Docs pass**: every crate README current; `docs/README.md` index updated;
   `docs/dev/*` (sdlc, stories, rendering) match reality; root README WIP
   statement updated to reflect the slice's actual capabilities; CHANGELOG-ish
   summary in the release notes of the slice tag.
3. **Acceptance walkthrough**: execute plan.md acceptance criteria 1–5
   explicitly; record evidence (URLs, command output, screenshots) in
   `_DONE.md`.
4. **Roadmap bookkeeping**: mark plan.md `status: done`; archive the planning
   directory to `docs/archive/plans/` per convention; seed the next planning
   cycle's candidate list (editing UI, keywords, stats...) into a
   `future.md`.

## Validation

`just ci` green; fresh-clone → `just serve-web` works following only README
instructions (test the newcomer path — have James do it).

## Agent reminders

Work on a branch; commit, push, PR, and fix CI autonomously (per
CONTRIBUTING.md). Fix-or-file, don't silently drop findings. The
"fresh clone" test must be honest — no relying on machine state.

## Definition of done

Acceptance criteria all pass with recorded evidence; repo clean; docs
truthful; plan archived with `_DONE.md`.
