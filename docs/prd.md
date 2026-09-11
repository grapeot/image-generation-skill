# Product Requirements Document

## Product

Image Generation Skill is a public-safe Python repo that lets AI agents generate, edit, and upscale images through a stable local CLI and importable library. The repo packages a previously local single-file utility into a reusable public skill without carrying private workspace assumptions.

## Users

The primary user is an AI coding or operating agent working inside a local workspace. The human user gives the agent a goal such as generating a hero image, editing an input image, or upscaling a draft artifact. The agent reads the root skill, runs the local command, and leaves image files on disk.

The secondary user is a developer who wants to vendor the skill into another workspace, run offline tests, and expose one root Markdown skill to their agent discovery chain.

## Goals

The repo must provide a stable `generate-image` command, importable Python functions, offline tests, public-safe docs, and one root skill file. It must support Gemini Flash, Gemini Pro, GPT-Image-2, GPT-Image-2.5 (Sunburst and Flare), Gemini image editing, OpenAI image editing with multiple reference images, and Gemini upscaling.

The public repo must be safe to publish. Examples use fake credentials, generic 1Password references, and no local absolute paths. Live API behavior must be opt-in through real local configuration.

## Non-Goals

This repo does not manage image asset catalogs, perform browser-based design review, choose prompts for a brand system, host generated outputs, or store private credentials. It also does not expose multiple global skills; the root skill is the routing surface.

## Functional Requirements

The CLI must accept text prompts, optional input images, output paths or prefixes, image size, aspect ratio, model selection, quality selection for OpenAI, and an upscale mode.

The package must expose `generate()` and `upscale()` as importable functions. Missing API keys should raise typed exceptions in library code rather than calling `sys.exit()`.

The CLI must return a nonzero exit code and a clear stderr message on configuration or validation failures.

## Safety Requirements

Default tests must not call Gemini or OpenAI. Live tests, if introduced later, must require `IMAGE_GENERATION_ENABLE_LIVE_TESTS=1` and real user-provided credentials.

The repo must not include private workspace paths, concrete private 1Password references, real API keys, generated user images, logs, or local data.

## Success Criteria

A fresh checkout can install with `uv`, run offline tests, render `generate-image --help`, and give an AI agent enough information in `skills/skill_image_generation.md` to generate or upscale images after the human configures real credentials.
