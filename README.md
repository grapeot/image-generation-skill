# Image Generation Skill

A public-safe Python skill repo for AI image generation and upscaling. It gives agents one stable interface, `generate-image`, for Gemini image generation, Gemini image upscaling, and OpenAI image generation.

This repo is designed for AI-assisted installation into a larger workspace. The package code lives under `src/image_generation_skill/`, the convenience wrapper is `scripts/generate-image`, and the only root skill exposed by this repo is `skills/skill_image_generation.md`.

## Privacy Statement

This repository is designed to be publishable with only fake examples. Public files use placeholder API keys, generic `op://your-vault/your-item/your-field` 1Password references, and example prompts only. It contains no private credentials, no private 1Password vault names, no workspace-specific absolute paths, and no generated user images.

## Install For Humans

```bash
git clone https://github.com/grapeot/image-generation-skill.git
cd image-generation-skill
cp .env.example .env
# Edit .env with your real API keys or your own generic 1Password references.
uv venv .venv
uv pip install --python .venv/bin/python -e '.[dev]'
```

Run the offline checks:

```bash
.venv/bin/python -m pytest -v
scripts/generate-image --help
```

Generate an image after configuration:

```bash
scripts/generate-image -p "A clean product photo of a ceramic coffee dripper" -o output/dripper.jpg
```

Upscale an image with Gemini:

```bash
scripts/generate-image --upscale -i input.jpg -o output/input_4k.jpg --aspect-ratio 16:9
```

## AI Installation Into A Workspace

Give this repo URL to Codex, Claude Code, Cursor, OpenCode, or another coding agent:

```text
https://github.com/grapeot/image-generation-skill
```

Ask the agent to install it into the target workspace. The agent should first read the target workspace's `AGENTS.md`, `CLAUDE.md`, or equivalent local instructions. If that workspace has a routing file such as `WORKSPACE.md`, the agent should follow it before choosing where to clone or vendor this repo.

The agent should then clone or vendor this repository into the target workspace, install it with `uv`, and expose exactly one root skill to the workspace discovery chain: `skills/skill_image_generation.md`. If the target workspace has `rules/skills/INDEX.md` or `skills/INDEX.md`, add one entry pointing to that root skill. If it has no skill index, add a short pointer in `AGENTS.md` or `CLAUDE.md` telling future agents to read `skills/skill_image_generation.md` for image generation and upscaling tasks.

Do not expose multiple internal files as global skills. This repo has one public root skill so future private overlays can add local aliases or credential guidance without mixing private details into the public implementation.

## Configuration

The CLI loads `.env` from the current project context through `python-dotenv`. It does not assume any external workspace root.

Required for Gemini generation or upscaling:

```text
GEMINI_API_KEY=replace-with-your-gemini-api-key
```

`GOOGLE_API_KEY` is accepted as a Gemini fallback. Required for OpenAI image generation:

```text
OPENAI_API_KEY=replace-with-your-openai-api-key
```

Optional model overrides are listed in `.env.example`. Generic 1Password references are supported by setting `GEMINI_API_KEY_1PASSWORD_REF` or `OPENAI_API_KEY_1PASSWORD_REF` to a value like `op://your-vault/your-item/your-field`.

## CLI Contract

```bash
generate-image -p "A serene mountain lake" -o lake.jpg
generate-image -p "A cinematic mountain lake" -o lake.jpg -m gemini-pro
generate-image -p "A product photo" -o product.png -m gpt-image-2 --quality medium
generate-image -p "Remove the background" -i photo.jpg -o clean.png
generate-image -p "Wide banner" -o banner.jpg --size 4K --aspect-ratio 16:9
generate-image --upscale -i small.jpg -o big.jpg
```

Supported model aliases are `gemini-flash`, `gemini-pro`, and `gpt-image-2`. Exact model IDs currently accepted by the CLI are `gemini-3.1-flash-image-preview`, `gemini-3-pro-image-preview`, and `gpt-image-2`.

OpenAI size aliases map to concrete pixel sizes. For example, `1K + 16:9` maps to `1536x864`, and `4K + 16:9` maps to `3840x2160`.

## Development

```bash
uv venv .venv
uv pip install --python .venv/bin/python -e '.[dev]'
.venv/bin/python -m pytest -v
scripts/generate-image --help
```

Default tests are offline and must not call Gemini or OpenAI. Live tests, if added later, must be skipped unless `IMAGE_GENERATION_ENABLE_LIVE_TESTS=1` is set.
