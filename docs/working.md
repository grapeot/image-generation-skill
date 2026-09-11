# Working Notes

## Changelog

### 2026-09-11

- Added GPT Image 2.5 model aliases: `gpt-image-2.5-sunburst` (precision-first) and `gpt-image-2.5-flare` (speed-first), both at the same price as `gpt-image-2`, which remains supported.
- Extended `--quality` to `low|medium|high|xhigh|max|auto` (GPT Image 2.5 adds `xhigh`, `max`, and `auto`); parser default stays `medium`.
- Removed the OpenAI single-input limit: every repeatable `-i` input is now passed to one `images.edit` call, matching GPT Image 2.5's native multi-reference support. The `edit` protocol type now accepts a sequence of binary streams.
- `--upscale` now rejects any `gpt-image*` model instead of only `gpt-image-2`.
- Updated `skills/skill_image_generation.md`, `README.md`, `.env.example`, `docs/prd.md`, `docs/rfc.md`, and `docs/test.md`.
- Added offline tests for the 2.5 aliases, the extended quality tiers, `--upscale` rejection, and multi-input editing through an injected fake OpenAI client. Verified `.venv/bin/python -m pytest -q` — 27 passed.

### 2026-05-25

- Replaced broad pyright diagnostic suppression with targeted provider protocols, typed argparse coercion, public test-facing helper aliases, and pyright/basedpyright excludes for `.venv`, caches, and build artifacts.
- Verified `lsp_diagnostics` reported no diagnostics for `src/image_generation_skill/core.py`, `src/image_generation_skill/cli.py`, and `tests/test_core.py`.
- Verified `.venv/bin/python -m pytest -v` still passed 23 offline tests and `scripts/generate-image --help` still rendered the same CLI contract.
- Root directory LSP scans still enumerate `.venv` despite repo-level pyright and basedpyright excludes; targeted file and `src/` / `tests/` scans are clean.
- Created the public-safe project scaffold with package code, CLI entry point, thin script wrapper, tests, docs, `.env.example`, `.gitignore`, `pyrightconfig.json`, and one root skill file.
- Ported the existing pure parser, model resolution, size mapping, output path, and validation test coverage into the package test suite.
- Refactored missing API key handling into typed exceptions so library calls do not call `sys.exit()`.
- Removed workspace-root `.env` assumptions and replaced concrete private 1Password defaults with optional generic `op://your-vault/your-item/your-field` references.
- Verified `lsp_diagnostics` with zero errors on `src/image_generation_skill/` and `tests/`.
- Verified `.venv/bin/python -m pytest -v` passed 23 offline tests.
- Verified `scripts/generate-image --help` and bad-input behavior through a tmux CLI session.
- Verified the required `.env.example` placeholders and ran a targeted privacy scan for private paths, concrete private 1Password refs, and common API key patterns.

## Lessons Learned

- Keep provider SDK imports lazy so offline tests can validate parser and configuration behavior without making API calls or requiring live credentials.
- Public skill repos should expose one root skill file and keep private credential routing in the installing workspace, not in the public package.
- Missing credential behavior belongs in typed library exceptions; the CLI is the only layer that should translate expected failures into process exit codes.
