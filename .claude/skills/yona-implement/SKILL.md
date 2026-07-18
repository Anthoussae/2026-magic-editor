---
name: yona-implement
description: Execute an existing repo-local planning artifact, validate the work, create ADRs when warranted, archive the completed plan, commit on a work branch, and hand off to yona-push for PR creation and CI.
---

# Yona Implement

Use this skill to execute an existing `plan.md` with disciplined validation.

## Plan Location

Plans usually live under:

```text
docs/plans/<YYYY-MM-DD>-<name>/plan.md
```

If the user provides an explicit plan path, use it. Otherwise:

1. Determine the repo root with `git rev-parse --show-toplevel`. If the project is not a git repo, use the current working directory.
2. Look under `docs/plans/` for active plans.
3. If several plans match, ask the user which one to execute.

Completed plans should be archived under:

```text
docs/archive/plans/<YYYY-MM-DD>-<name>/
```

## Setup

Read `plan.md` first. Inspect:

- `depth`
- phase/work-item files such as `01-*.md`
- phase directories with their own `plan.md`
- ADR expectations/candidates
- documentation expectations, affected docs, or missing docs called out by the plan
- validation commands
- existing `_DONE.md`, if present

If the plan is missing phase files but marks a work item as `sub-phases required`, write those sub-phase files before implementation.

If `_DONE.md` already exists for the requested plan, inspect it and the current plan status before continuing. Ask before re-running completed work unless the user explicitly requested a follow-up or repair.

Before touching code, make sure you are on a work branch. If the current branch is `main` (or another protected base branch), create a descriptively named branch first (e.g. `m1-repo-foundation`). Never implement directly on `main`.

## Execution

For each work item:

1. Execute directly or delegate to another agent when appropriate and available.
2. Keep the work inside the stated scope.
3. Review every delegated result before moving on.
4. Re-run the work-item validation commands.
5. Record phase-level implementation results when phase files exist.
6. Update relevant documentation when the change alters public behavior, workflows, architecture, commands, package/module purpose, configuration, operations, or planning/process conventions.
7. Check for shortcuts: TODOs, stubbed logic, suppressed warnings, disabled tests, scope creep, stale docs, or unrelated refactors.

If a phase says `sub-phases recommended`, decide whether to split before execution. If it says `sub-phases required`, split before execution.

Delegated agents should not commit; the implementing agent owns commits and makes them at validated checkpoints.

When recording phase-level implementation results, append a short section to the phase file instead of renaming it:

```md
## Implementation Result

Status: done
Completed: YYYY-MM-DD
Commit: pending

- Changed: ...
- Validated: ...
- Deviations: none
```

After the final commit exists, update `Commit: pending` entries with the commit SHA when practical.

## Stop And Ask

Pause before continuing when:

- The plan is ambiguous or contradictory.
- Validation fails twice for the same work item.
- Fixing an issue would expand scope or change public APIs beyond the plan.
- A hard bug appears that needs real debugging.
- The requested plan path cannot be resolved.

## ADRs

Do not write `summary.md`.

After implementation and validation, evaluate ADR candidates against the completed diff.

Create accepted ADR files in the repo when a decision meets this criterion:

> The change chooses a direction among plausible alternatives and that choice has lasting architectural, operational, security, data-model, API, workflow, or cross-repo/process consequences.

Use:

```text
docs/adr/YYYY-MM-DD-short-title.md
```

If no ADR is warranted, state that in the final response.

Historical ADR backfill is different: produce an ADR candidate list for human review before creating historical ADR files unless the user explicitly approved the candidates.

## Documentation

Before final validation and any commit, review the completed diff for documentation impact.

Update docs when the implementation changes:

- public commands, setup, deployment, validation, or troubleshooting steps
- package/module purpose, especially `README.md` files next to changed packages
- architecture, runtime boundaries, module ownership, data models, APIs, or workflows
- user-visible behavior, UI flows, permissions, operational runbooks, or process conventions
- ADR-worthy decisions that should be durable in `docs/adr/`

Prefer existing repo docs near the changed code before adding new docs. If no docs need changes, record why in the implementation log or final response. If docs are intentionally deferred, record the follow-up explicitly.

## Implementation Log

After implementation, validation, ADR handling, and any requested commit, write `_DONE.md` in the planning directory. This is the completion log for what actually happened, separate from ADRs and separate from `notes.md`.

Use frontmatter:

```md
---
kind: implementation-log
status: done
repo: <repo-slug>
plan: plan.md
completed: YYYY-MM-DD
commit: <sha-or-none>
adrs:
  - docs/adr/...
---
```

Then include:

- Outcome.
- Completed work.
- Validation commands and results.
- Deviations from the plan, or `None`.
- Documentation updated, or why no docs were needed.
- ADRs created, or why no ADR was warranted.
- Follow-up work that remains outside the completed scope.

Update `plan.md` frontmatter to `status: done` and add `completed: YYYY-MM-DD` and `commit: <sha-or-none>`.

Do not rename `plan.md` or phase files after completion.

## Commit

Committing is the default — do not wait for the user to ask. Once validation passes, create a single conventional commit on the work branch, unless the plan explicitly calls for checkpoint commits (then commit at each validated checkpoint).

Commit code, plan artifacts, and ADRs together. Include ADR paths in the commit body when ADRs were created.

Only skip committing if the user explicitly asked you not to commit, or if validation never passed — in that case leave the worktree ready for review and clearly report the changed files and validation results.

## Archive

After `_DONE.md` is written and the implementation commit exists, move the completed planning directory from:

```text
docs/plans/<YYYY-MM-DD>-<name>/
```

to:

```text
docs/archive/plans/<YYYY-MM-DD>-<name>/
```

If a destination already exists, add a short suffix such as `-v2` rather than overwriting it.

Include the archive move in the implementation commit when possible. If the move happens after the commit, commit it as a small follow-up on the same branch.

## Hand Off To Push

After the implementation commit and archive are done, continue directly into the `yona-push` workflow: push the branch, create or update the PR, watch CI, and repair failures until green. Do not stop to ask permission for this step — the loop's human touchpoint is the finished PR.

Skip the handoff only if the user explicitly said not to push, if there is no commit, or if a milestone review gate says to stop earlier.

## Completion

Finish with:

- Outcome.
- Files changed.
- Validation commands and results.
- Documentation updated, or why no docs were needed.
- ADRs created, or why none were warranted.
- Archive path.
- Commit SHA(s).
- PR link and CI state (from the `yona-push` handoff).
- Any remaining follow-up work, and what the human should review before merging.

## Repo Conventions — 2026-magic-editor

This section customizes the skill for this repo. It overrides the generic text above where they differ.

- **Autonomous loop**: implement → commit → push → PR → CI-green is one continuous agent-driven flow (see `CONTRIBUTING.md`). Humans review at the PR and do the merge. Milestone review gates still apply where a milestone file declares one.
- **Commit style**: imperative subject ≤72 chars; body explains *why*; reference the milestone/plan (e.g. `[M4]`); agent-authored commits end with `Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>`. Commit code + plan artifacts + ADRs together (as the generic text says).
- **Validation baseline**: `just ci` must pass before declaring a work item done (fmt, clippy with warnings denied, tests, wasm build). Do not weaken lints to pass.
- **Licensing discipline (check on every new file)**:
  - Ported code (`mse-format`, `mse-script`, `mse-model`, ported parts of `mse-render`): `// SPDX-License-Identifier: GPL-3.0-or-later` + `// Ported from MagicSetEditor2 <original path>`.
  - Original code (`app-web`, stories, tools, original engine files): `// SPDX-License-Identifier: AGPL-3.0-only`.
  - Porting from a new area of MSE2 for the first time → check `NOTICE` still describes the derivation accurately.
- **Engine purity**: `mse-*` crates must stay host-testable — no dioxus/web-sys/wasm-only deps. If a work item seems to need one, stop and ask.
- **Stories**: from M2 onward, UI components do not land without stories covering their main states. Treat a missing story like a missing test.
- **Asset/IP policy**: never commit, fetch, bundle, or hot-link third-party template assets (only our original demo template assets, attributed in `assets/ATTRIBUTION.md`). Local fixtures live in gitignored `fixtures/local/`; tests against them must skip cleanly when absent.
- **Teaching**: when implementation introduces a new concept/workflow, update or create the matching `docs/dev/` page — the repo should explain itself to James.
