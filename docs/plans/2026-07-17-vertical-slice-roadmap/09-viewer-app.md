# M9 — Viewer App: Lists, Display, User File Import

- **ID**: M9 · **Size**: md · **Review gate**: end-of-milestone (UX review)
- **Dependencies**: M2 (stories), M7, M8. M5 demo set for bundling.
- **ADR expectation**: none expected (OPFS layout decision goes in the ADR
  only if it constrains future sync/backend work).

## Scope

The user-facing slice, story-first per M2 workflow:

1. **Bundled demo**: app ships the M5 demo set (our original assets — the
   only assets we may bundle); landing page shows its card list immediately.
2. **Card list + card view**: list (name + sort field) → select → rendered
   card (M8) with zoom presets; basic keyboard navigation.
3. **User file import**: drag-drop / file-picker for `.mse-set` and template
   packages (zip). Missing-template flow: a set referencing an unavailable
   template gets a clear message telling the user to provide that package
   file too (never a download link *in the app*; docs may link to community
   sources). Imported packages persist in OPFS (adapt `lpa-fs-opfs`
   patterns from lp2025).
4. **IP posture in UI**: user-provided content stays local (OPFS); a short
   "your files never leave your browser" note in the import UI.
5. Every component gets stories (empty states, long names, missing images,
   load errors).

Out of scope: editing, set management beyond import/delete, sharing/URLs,
mobile polish.

## Validation

`just ci` green (incl. wasm); stories cover the components; manual E2E on the
deployed site: demo set browses; a real community set (user-provided, local
test only) imports and renders.

## Review gate (end of milestone)

James reviews the deployed slice UX and the import flow wording
(legal-adjacent copy).

## Agent reminders

Work on a branch; commit, push, PR, and fix CI autonomously (per
CONTRIBUTING.md). Story-first — no component lands without
stories. Do not add any asset fetching. Report changes, validation,
deviations.

## Definition of done

Deployed Pages site fulfills acceptance criteria 1–2 of plan.md; review gate
passed.
