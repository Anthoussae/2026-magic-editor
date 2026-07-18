# M5 — Original Demo Template + Fixtures Pipeline

- **ID**: M5 · **Size**: md · **Review gate**: end-of-milestone (asset review)
- **Dependencies**: M4 (format), usable before M6–M8 finish; grows alongside
  them. Parallelizable with M6.
- **ADR expectation**: expected (ADR: asset/IP separation policy — codifies
  the user-provided-file model decided in planning).

## Scope

Two-track fixtures per the decided IP policy (see plan.md + notes.md):

1. **Original demo template** (in-repo, CI-safe): author a small card game
   template in MSE2 format — `demo.mse-game`, a stylesheet, a symbol font —
   with **entirely original** names, art, frames, and symbols (AGPL-3.0 like
   the rest of our original work). Plus a small demo `.mse-set` using it.
2. **Local real-template fixtures** (never committed): `fixtures/local/`
   (gitignored) + `fixtures/README.md` documenting how a developer obtains
   community templates (link to https://magicseteditor.boards.net additional
   downloads board) and MSE2 itself for golden exports. Compat tests
   auto-skip when fixtures are absent.

Out of scope: rendering the demo template (M8), app bundling (M9).

## Work items

1. Design the demo game small but engine-exercising: text fields (styled,
   symbol-font cost field), a choice field with per-choice images (frame
   color), an image field, a computed field (script; lands with M6/M7),
   shrink-to-fit rules text. Keep scripts minimal and readable — this doubles
   as teaching material and as our compat test template.
2. Produce original assets: simple frame art, 4–6 cost/stat symbols, an OFL
   font. Document provenance of every asset in `assets/ATTRIBUTION.md`
   (AI-generated art is acceptable; note it).
3. Validate the packages load in **real MSE2** (Windows VM or Wine, or ask a
   Windows-having friend; worst case defer to golden comparison in M8) — the
   demo template is only a fair test if MSE2 itself accepts it.
4. `fixtures/local/` convention + skip-if-absent test helper in a small
   `test-support` crate; `fixtures/README.md` with the manual download steps
   and the legal rationale (no redistribution; user-provided only).
5. ADR: asset/IP separation policy (repo, CI, runtime, deployment).

## Review gate (end of milestone)

James reviews: demo template design + art direction, `ATTRIBUTION.md`, the
fixtures README wording (it makes legal claims), and the ADR.

## Agent reminders

Work on a branch; commit, push, PR, and fix CI autonomously (per
CONTRIBUTING.md). Nothing WotC-derived in any committed asset —
when in doubt, leave it out and flag. Original ≠ parody: don't imitate magic
frame trade dress. Stop and report if MSE2 validation (item 3) is infeasible.

## Definition of done

Demo packages parse with `mse-format`; assets attributed; fixtures pipeline
documented; ADR written; review gate passed.
