# RFC: Public Image Generation Skill

## Summary

The repo turns a local image generation script into a public package with a stable CLI and one root skill. The implementation keeps the useful model routing and size mapping behavior while removing workspace-root `.env` assumptions and private 1Password defaults.

## Architecture

`src/image_generation_skill/core.py` contains provider selection, API key resolution, argument validation helpers, output path construction, Gemini generation, OpenAI generation, and Gemini upscaling. `src/image_generation_skill/cli.py` is a thin argparse entry point that calls the library and converts exceptions into process exit codes.

`scripts/generate-image` is a shell wrapper for local checkout usage. It prefers the installed console command inside `.venv` and falls back to `PYTHONPATH=src python -m image_generation_skill.cli`.

`skills/skill_image_generation.md` is the single public root skill. Workspace-specific overlays may point to this file, but private aliases and credentials stay outside the public repo.

## Configuration Model

The package calls `python-dotenv` without a hardcoded path. This loads `.env` from the current project context rather than assuming a larger workspace layout.

Gemini credentials resolve from `GEMINI_API_KEY`, then `GOOGLE_API_KEY`, then optional generic 1Password reference variables. OpenAI credentials resolve from `OPENAI_API_KEY`, then optional generic 1Password reference variables. The implementation has no concrete vault, item, or field defaults.

## Provider Routing

`gemini-flash` maps to `gemini-3.1-flash-image-preview`. `gemini-pro` maps to `gemini-3-pro-image-preview`. `gpt-image-2`, `gpt-image-2.5-sunburst`, and `gpt-image-2.5-flare` map to OpenAI (Sunburst is precision-first, Flare is speed-first). Environment variables can override the concrete model IDs while the CLI keeps a small set of accepted public aliases.

OpenAI editing passes every repeatable `-i` input to a single `images.edit` call. GPT Image 2.5 accepts multiple reference images natively, so the CLI imposes no single-input limit.

OpenAI uses deterministic size mapping because the provider expects concrete pixel sizes. Gemini receives `image_size` and optional `aspect_ratio` through `types.ImageConfig`, which requires modern `google-genai` versions.

## Error Handling

Library functions raise typed exceptions such as `MissingApiKeyError`, `DependencyError`, and `ImageGenerationError`. The CLI catches expected exceptions, prints a short stderr message, and returns exit code 1. Parser validation still uses argparse errors, which produce standard `SystemExit` behavior for invalid command syntax.

## Public Boundary

The repo can be published as-is because public files use placeholders and generic examples. `.gitignore` blocks `.env`, `.env.*`, build outputs, caches, logs, data, and generated image directories while preserving `.env.example`.

## Testing Strategy

Default tests cover parser behavior, model resolution (including the GPT Image 2.5 aliases and the extended quality tiers), OpenAI size mapping, output path construction, argument validation, missing key exceptions, OpenAI multi-input editing through a fake client, and CLI nonzero behavior. These tests do not import Gemini or OpenAI SDKs for network work and do not call provider APIs.
