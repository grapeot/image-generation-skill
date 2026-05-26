# Image Generation Skill

## What this repo is

This repository packages a public-safe image generation skill for AI agents. It provides an importable Python library, a stable `generate-image` console command, a thin `scripts/generate-image` wrapper, offline tests, and one root skill document at `skills/skill_image_generation.md`.

The project supports Gemini image generation, Gemini image upscaling, and OpenAI image generation through explicit API keys supplied by the user. It does not ship private credentials, workspace-specific paths, or private 1Password item names.

## Working environment

Use a project-local `.venv` created with `uv` when changing or testing the package.

```bash
uv venv .venv
uv pip install --python .venv/bin/python -e '.[dev]'
.venv/bin/python -m pytest -v
scripts/generate-image --help
```

The default test suite must stay offline. Live API tests, if added later, must require `IMAGE_GENERATION_ENABLE_LIVE_TESTS=1` and valid fake-to-real `.env` replacement by the user.

## Code boundaries

`src/image_generation_skill/` is the only package logic layer. Keep network calls behind explicit generation or upscale functions. Import Gemini and OpenAI SDKs lazily so parser tests and configuration checks can run without contacting external services.

`skills/skill_image_generation.md` is the only root skill exposed by this repo. Do not add additional globally exposed skill files unless this root document routes to them.

## Public safety

All public docs, tests, and examples must use fake credentials and generic 1Password references such as `op://your-vault/your-item/your-field`. Do not add private API keys, private vault names, private domains, internal paths, downloaded images, generated artifacts, or local data to the repo.

Before handing off public-facing changes, run the offline tests and a privacy scan for private paths and concrete secret references. Update `docs/working.md` with meaningful validation results.
