# Image Generation Skill

## When To Use

Use this skill when the user asks an AI agent to generate images, edit an image with a prompt, upscale an existing image, or choose between Gemini and OpenAI image models for a local artifact-producing workflow.

This skill produces local image files. It is not a design review process, a stock image search workflow, or a private credential guide.

## Working Directory And Entry Points

Run commands from the repository root, next to `pyproject.toml`.

Preferred installed command:

```bash
generate-image --help
```

Repository wrapper:

```bash
scripts/generate-image --help
```

Module form:

```bash
python -m image_generation_skill.cli --help
```

## Configuration

Use `.env.example` as the public template. Fill a local `.env` with real values before making live API calls.

Gemini uses `GEMINI_API_KEY`, with `GOOGLE_API_KEY` accepted as a fallback. OpenAI uses `OPENAI_API_KEY`. Optional generic 1Password references can be supplied through `GEMINI_API_KEY_1PASSWORD_REF` and `OPENAI_API_KEY_1PASSWORD_REF`, using the form `op://your-vault/your-item/your-field`.

The package loads `.env` from the current project context. It does not depend on any external workspace root.

## Core Commands

Text-to-image with default Gemini Flash:

```bash
generate-image -p "A serene mountain lake at sunset" -o output/lake.jpg
```

Gemini Pro:

```bash
generate-image -p "A cinematic mountain lake" -o output/lake.jpg --model gemini-pro
```

OpenAI image generation:

```bash
generate-image -p "A clean product photo on a white background" -o output/product.png --model gpt-image-2 --quality medium
```

Image editing with prompt and one input:

```bash
generate-image -p "Remove the background and keep the subject natural" -i input/photo.jpg -o output/clean.png
```

Gemini image editing with multiple inputs:

```bash
generate-image -p "Combine the first image composition with the second image color palette" -i input/layout.jpg -i input/style.jpg -o output/combined.jpg
```

4K generation with aspect ratio:

```bash
generate-image -p "Wide hero banner for a technical article" -o output/banner.jpg --size 4K --aspect-ratio 16:9
```

Upscale an existing image with Gemini:

```bash
generate-image --upscale -i input/small.jpg -o output/small_4k.jpg --aspect-ratio 16:9
```

## Model And Size Controls

Supported model aliases are `gemini-flash`, `gemini-pro`, and `gpt-image-2`. Exact accepted IDs are `gemini-3.1-flash-image-preview`, `gemini-3-pro-image-preview`, and `gpt-image-2`.

Environment variables can override model IDs: `IMAGE_GENERATION_MODEL`, `GEMINI_FLASH_IMAGE_MODEL`, `GEMINI_IMAGE_GENERATION_MODEL`, `GEMINI_PRO_IMAGE_MODEL`, `OPENAI_IMAGE_MODEL`, `GEMINI_IMAGE_UPSCALE_MODEL`, and `IMAGE_UPSCALE_MODEL`.

OpenAI size mappings are deterministic. Examples: `1K + 1:1` maps to `1024x1024`, `1K + 16:9` maps to `1536x864`, and `4K + 16:9` maps to `3840x2160`.

`--quality low|medium|high` applies to `gpt-image-2`. Gemini accepts the flag but ignores it because quality tiers are provider-specific.

## Agent Safety Rules

Default tests are offline. Do not run live Gemini or OpenAI calls during installation or validation unless the user explicitly configured real API keys and asked for a live generation.

Never write credentials into prompts, command history summaries, docs, tests, or generated artifacts. Public examples should use fake API keys and generic 1Password references only.

Generated images, logs, and local data belong in ignored directories such as `output/`, `generated/`, `logs/`, or `data/`.

## Acceptance Criteria

A local installation is ready when `generate-image --help` renders successfully, `python -m pytest -v` passes offline, `.env` is local and ignored by git, and the workspace exposes only this root skill file for image generation discovery.
