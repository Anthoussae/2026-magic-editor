# Magic Demo — a web-based card set editor

A browser-based editor for designing custom trading cards, inspired by
[Magic Set Editor 2](https://github.com/twanvl/MagicSetEditor2) (MSE2), the
long-standing Windows desktop app.

Goals for the initial phase:

- **Frontend-only**: no backend; deployable to GitHub Pages; persistence via
  browser storage (with file import/export).
- **Well-tested, modular, hexagonal architecture** with anti-corruption layers
  around external concerns (storage, rendering targets, legacy MSE formats).
- **Component-based, story-driven UI** and data-driven UI where the domain
  (field/style templates) naturally calls for it.

## Documentation

- [docs/](docs/) — analysis of the original MSE2 codebase, and design
  documents for this project.

## Status

Early exploration. No application code yet.
