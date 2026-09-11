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

OpenAI image generation (GPT Image 2.5):

```bash
generate-image -p "A clean product photo on a white background" -o output/product.png --model gpt-image-2.5-sunburst --quality high
```

GPT Image 2.5 ships as two variants at the same price: `gpt-image-2.5-sunburst` (precision-first, for final assets) and `gpt-image-2.5-flare` (speed-first, for fast drafts). `gpt-image-2` remains available.

Final delivery format rule for external Markdown deliverables: every embedded image in a survey report, blog draft, course material, or share-ready document must be a PNG, JPG, or WebP file produced by a GPT Image model (`gpt-image-2.5-sunburst` / `gpt-image-2.5-flare` / `gpt-image-2`) generation/redraw, or a compressed derivative of that output. SVG, Mermaid, matplotlib, Pillow, HTML/CSS screenshots, Graphviz, draw.io exports, Keynote/PowerPoint exports, and other local/generated visuals are allowed only as intermediate editable sources or structure drafts. They must not be embedded directly, even if converted to PNG/JPG/WebP locally. Local conversion or compression does not satisfy the final-delivery rule unless the visual has passed through a GPT Image model. SVG rendering differs across Markdown renderers, mobile clients, publishing pipelines, security policies, fonts, sizing, and dark-mode inheritance.

Image editing with prompt and one input:

```bash
generate-image -p "Remove the background and keep the subject natural" -i input/photo.jpg -o output/clean.png
```

Image editing with multiple inputs (`-i` is repeatable; Gemini and GPT Image 2.5 both accept several inputs):

```bash
generate-image -p "Combine the first image composition with the second image color palette" -i input/layout.jpg -i input/style.jpg -o output/combined.jpg
generate-image -p "Merge the logo and the QR code into one closing card" -i input/logo.png -i input/qr.png -o output/closing.png --model gpt-image-2.5-sunburst
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

Supported model aliases are `gemini-flash`, `gemini-pro`, `gpt-image-2`, `gpt-image-2.5-sunburst`, and `gpt-image-2.5-flare`. Exact accepted IDs are `gemini-3.1-flash-image-preview`, `gemini-3-pro-image-preview`, `gpt-image-2`, `gpt-image-2.5-sunburst`, and `gpt-image-2.5-flare`.

Environment variables can override model IDs: `IMAGE_GENERATION_MODEL`, `GEMINI_FLASH_IMAGE_MODEL`, `GEMINI_IMAGE_GENERATION_MODEL`, `GEMINI_PRO_IMAGE_MODEL`, `OPENAI_IMAGE_MODEL`, `GEMINI_IMAGE_UPSCALE_MODEL`, and `IMAGE_UPSCALE_MODEL`.

OpenAI size mappings are deterministic. Examples: `1K + 1:1` maps to `1024x1024`, `1K + 16:9` maps to `1536x864`, and `4K + 16:9` maps to `3840x2160`.

`--quality low|medium|high|xhigh|max|auto` applies to GPT Image models. GPT Image 2.5 adds `xhigh`, `max`, and `auto`, and its quality scale is finer than GPT Image 2's: 2.5 `high` costs about a quarter of GPT Image 2 `high`, and 2.5 `max` matches the old `high`. Gemini accepts the flag but ignores it because quality tiers are provider-specific.

## Agent Safety Rules

Default tests are offline. Do not run live Gemini or OpenAI calls during installation or validation unless the user explicitly configured real API keys and asked for a live generation.

Never write credentials into prompts, command history summaries, docs, tests, or generated artifacts. Public examples should use fake API keys and generic 1Password references only.

Generated images, logs, and local data belong in ignored directories such as `output/`, `generated/`, `logs/`, or `data/`.

## Known Caveats

These pitfalls were observed in real agent workflows and are worth checking before invoking GPT Image 2.5 or local chart libraries.

- **GPT Image timeout**: the default bash timeout (commonly 120s) is too short for GPT Image 2.5. Complex image edits such as redrawing a matplotlib chart into an infographic routinely need 300s or more. Set an explicit timeout (for example `timeout=300000`, roughly 5 minutes). Without an explicit timeout, the command produces no output and exits on timeout.
- **Parallel multi-image generation**: GPT Image 2.5 has high per-call latency. When producing a set of hero images, covers, or variants, do not run them serially. Use Python `ThreadPoolExecutor` or shell concurrency to launch multiple `generate-image --model gpt-image-2.5-flare --quality low` jobs at once. An empirical sweet spot is 4-6 concurrent jobs with a per-task timeout around 420s. Write all outputs into an ignored/temp directory and verify every output path exists after the batch finishes.
- **matplotlib CJK rendering**: on macOS, matplotlib cannot render Chinese with Arial. Use `matplotlib.font_manager` to locate a system CJK font (`STHeiti`, `Heiti SC`, `PingFang HK`) and pass it via `FontProperties(fname=...)` per text element. Do not put font names directly into `rcParams['font.sans-serif']`; `font_manager.findfont()` is unreliable for CJK fonts on macOS.
- **GPT Image is repaint-only, not fresh generation**: when building infographics with GPT Image 2.5, first produce a matplotlib structure draft and pass it as the `-i` input image. The model understands the draft layout, preserves title and annotation text, and improves the visual presentation. Asking it to generate an infographic from a text prompt alone (no input image) produces unreliable results. `--aspect-ratio 16:9` suits horizontal infographics.

## Acceptance Criteria

A local installation is ready when `generate-image --help` renders successfully, `python -m pytest -v` passes offline, `.env` is local and ignored by git, and the workspace exposes only this root skill file for image generation discovery.
