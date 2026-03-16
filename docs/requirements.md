# Bubble — Product Requirements

## Overview

Bubble is a bulk image processing web application. Users upload multiple images, configure processing options via a single form, and download all processed images as a single ZIP file.

The application is anonymous (no accounts), stateless (no database), and server-side only (all image processing happens on the backend).

## Supported Formats

- **Input**: JPEG, PNG, WebP
- **Output**: JPEG, PNG, WebP (including cross-format conversion)
- Users may upload a mix of different formats in a single batch.

## Upload Limits

- **Maximum images per batch**: 50
- **Maximum total upload size**: TBD (depends on Express/deployment constraints, but target ~100–200 MB)

## Processing Operations

All operations are applied uniformly to every image in the batch. The user interface is a single flat form (not a pipeline builder).

### Resize

- By percentage (e.g., 50%)
- By explicit dimensions (width and/or height in pixels)
- Maintain aspect ratio by default

### Crop

- Predefined aspect ratios (e.g., 1:1, 4:3, 16:9, 3:2)
- Crop anchor: center (no per-image manual cropping)

### Rotate

- Fixed angles: 90°, 180°, 270°

### Auto-Rotate (EXIF)

- Detect EXIF orientation metadata and rotate the image to its correct upright position
- Enabled by default (user can toggle off)

### Adjust Quality / Compression

- Quality slider (e.g., 1–100) for lossy formats (JPEG, WebP)
- PNG compression level

### Format Conversion

- Convert all images to a chosen output format (JPEG, PNG, or WebP)
- Option to keep original format

### Brightness & Contrast

- Brightness adjustment (e.g., -100 to +100)
- Contrast adjustment (e.g., -100 to +100)

### Watermark

- Text-based watermark overlay
- Configurable: text content, position (e.g., bottom-right), opacity, font size

### Strip Metadata

- Option to remove all EXIF/metadata from output images
- Off by default

## Output

- All processed images are packaged into a **single ZIP file** for download.
- Original filenames are preserved (with extension changed if format conversion is applied).
- If filename collisions occur (e.g., `photo.jpg` and `photo.png` both converted to WebP), append a numeric suffix.

## Preview (Phase 2)

- After uploading, the user sees a thumbnail of the first image.
- A "Preview" button sends the first image to the server with the current settings and returns the processed result.
- This gives the user visual confirmation before committing to a full batch process.
- **This feature is not part of the initial working version.** The first milestone delivers upload → configure → process → download without preview.

## Progress Feedback

- During batch processing, the user sees real-time progress (e.g., "Processing 12/50...").
- Communication method: Server-Sent Events (SSE) or similar mechanism (architecture decision).

## Rate Limiting (Future)

- Not part of MVP.
- Future plan: limit the number of requests per IP address per day.

## Non-Goals (Explicitly Out of Scope)

- User accounts / authentication
- Image persistence / storage / gallery
- Client-side image processing
- Per-image individual settings
- Adding/editing metadata (e.g., copyright fields) — may revisit later

## Project Goals

This project has two equally important goals:

1. **Build a working, real web application** — a usable bulk image processor with potentially real users.
2. **Learn AI-assisted multi-agent development** — the development process uses 6 independent AI agents (see `docs/agents.md`), coordinated by the human developer. Understanding and mastering this workflow is a primary purpose of the project.

## Development Workflow Decisions

- **Version control**: Manual Git workflow on `main`. No GitHub Issues or Pull Requests — task tracking lives in `docs/tasks.md`.
- **Pace**: Intentionally slow. Every line of code is reviewed and understood by the human developer.
- **Dependencies**: Zero frontend dependencies. Backend uses only the necessary packages (Express, Sharp, Multer, Archiver).
