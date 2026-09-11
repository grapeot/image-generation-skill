# Test Strategy

## Scope

The default suite verifies local contracts that can run without external services: CLI parsing, model alias resolution (including the GPT Image 2.5 Sunburst/Flare aliases), environment override behavior, the extended quality tiers (`low|medium|high|xhigh|max|auto`), OpenAI size mapping, output path construction, argument validation, missing API key handling, and OpenAI multi-input editing through an injected fake client.

## Offline Tests

Run offline tests with:

```bash
.venv/bin/python -m pytest -v
```

These tests must not call Gemini or OpenAI. They should stay safe for a new contributor who copied `.env.example` but has no real credentials.

## Live Tests

Live tests are not part of the default suite. If they are added later, they must use the `live_integration` pytest marker and skip unless `IMAGE_GENERATION_ENABLE_LIVE_TESTS=1` is set.

Live tests may verify real text-to-image generation, image editing, and upscaling. They should write outputs only to ignored directories such as `output/` or `generated/` and should never commit generated images.

## Manual QA

A release-ready checkout should satisfy these checks:

```bash
scripts/generate-image --help
generate-image --help
.venv/bin/python -m pytest -v
```

For live manual QA, run exactly one small generation only after the user has configured real credentials:

```bash
scripts/generate-image -p "A simple blue square icon" -o output/live_smoke.png --model gemini-flash
```

The live check is opt-in because it makes a network API call and may incur provider cost.

## Privacy Checks

Before publication, scan public files for private paths, concrete private 1Password references, and likely secrets. The expected repo state has no private workspace paths and only generic `op://your-vault/your-item/your-field` examples.
