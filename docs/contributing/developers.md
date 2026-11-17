# Developer Guide

General guidance for contributing to Gripora's firmware and configurator.

## Repositories

| Repo | Purpose |
|---|---|
| [gripora-firmware](https://github.com/gripora-project/gripora-firmware) | Flight controller firmware (C) |
| [gripora-configurator](https://github.com/gripora-project/gripora-configurator) | Configurator app (Svelte/Vite) |
| [gripora-docs](https://github.com/gripora-project/gripora-docs) | This documentation site (MkDocs) |
| [gripora-artifacts](https://github.com/gripora-project/gripora-artifacts) | Mirrored build artifacts (firmware binaries, etc.), used so the web Configurator can download them without hitting GitHub's release-asset CORS restrictions |
| [gripora-targets](https://github.com/gripora-project/gripora-targets) | Unified target board configs |

## Firmware changes

See [Building the Firmware](building-the-firmware.md) to get a local build
working first. Firmware PRs should build cleanly via `make unified` and
should not silently change saved-parameter layout without a parameter-group
version bump (see `PG_REGISTER` version numbers in `src/main/pg/`).

## Configurator changes

The Configurator is a Vite + Svelte app. See its repository's README for
local dev setup (`pnpm install`, `pnpm start`).

## Docs changes

This site is built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/).
Every page is plain Markdown under `docs/`; the site nav is defined in
`mkdocs.yml`. To preview locally:

```
pip install -r requirements.txt
mkdocs serve
```
