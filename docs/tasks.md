# Bubble — Task List

This file is the source of truth for what needs to be done. Each task is a single, commit-sized unit of work.

**Status legend**: `[ ]` not started · `[~]` in progress · `[x]` completed

---

## Phase 0: Project Setup

- [ ] **T001** — Initialize npm project with `package.json` (ES modules, scripts for dev/start/test)
- [ ] **T002** — Create folder structure (`client/`, `server/`, `tests/`, `docs/`)
- [ ] **T003** — Set up Express server skeleton (`server/index.js`, `server/app.js`) that serves `client/` as static files
- [ ] **T004** — Create minimal `client/index.html` with a "Hello Bubble" heading to verify static serving
- [ ] **T005** — Set up Vitest configuration
- [ ] **T006** — Set up Playwright configuration with a basic smoke test (page loads, heading visible)

## Phase 1: File Upload

- [ ] **T007** — Configure Multer middleware for multipart file uploads (accept JPEG, PNG, WebP; enforce file count and size limits)
- [ ] **T008** — Create `POST /api/process` route stub — accepts files + settings JSON, generates job ID (UUID), responds with 202
- [ ] **T009** — Write unit tests for file validation (correct MIME types, reject invalid files, enforce limits)
- [ ] **T010** — Build upload UI: file input, file list display, basic form layout
- [ ] **T011** — Wire upload UI to `POST /api/process` via fetch, display job ID in console

## Phase 2: State Machine

- [ ] **T012** — Implement generic state machine (`client/js/state-machine.js`) — define states, transitions, listeners
- [ ] **T013** — Write unit tests for generic state machine (transitions, invalid transitions, listeners fire)
- [ ] **T014** — Define Bubble app states and transitions (`client/js/app-machine.js`): idle → selected → processing → complete/error → idle
- [ ] **T015** — Connect state machine to UI (`client/js/ui.js`) — show/hide sections based on current state

## Phase 3: Settings Form

- [ ] **T016** — Build settings form HTML: all processing options with appropriate input types (sliders, checkboxes, dropdowns, number inputs)
- [ ] **T017** — Implement settings form JS: collect form values into the settings JSON schema
- [ ] **T018** — Client-side validation (`client/js/validation.js`): validate settings (e.g., dimensions > 0, quality 1–100, watermark text not empty when enabled)
- [ ] **T019** — Write unit tests for settings validation

## Phase 4: Image Processing (Core)

- [ ] **T020** — Implement `image-processor.js` — auto-rotate (EXIF)
- [ ] **T021** — Implement `image-processor.js` — rotate (fixed angles)
- [ ] **T022** — Implement `image-processor.js` — crop (predefined aspect ratios, center anchor)
- [ ] **T023** — Implement `image-processor.js` — resize (percentage and explicit dimensions)
- [ ] **T024** — Implement `image-processor.js` — brightness and contrast
- [ ] **T025** — Implement `image-processor.js` — watermark (text overlay)
- [ ] **T026** — Implement `image-processor.js` — strip metadata
- [ ] **T027** — Implement `image-processor.js` — format conversion and quality setting
- [ ] **T028** — Implement `image-processor.js` — orchestrator function that applies enabled operations in the correct order
- [ ] **T029** — Write unit tests for each processing operation (test with sample images)
- [ ] **T030** — Write integration tests for the orchestrator (multiple operations combined)

## Phase 5: Batch Processing & ZIP

- [ ] **T031** — Implement `zip-builder.js` — create ZIP from a directory of processed images using Archiver
- [ ] **T032** — Write unit tests for ZIP builder (correct files included, filenames preserved)
- [ ] **T033** — Implement batch processing in the process route — loop through uploaded images, process each sequentially, write to output directory
- [ ] **T034** — Handle filename collisions (duplicate names after format conversion → append numeric suffix)
- [ ] **T035** — Implement `GET /api/download/:jobId` — serve the ZIP file
- [ ] **T036** — Write integration tests for the full upload → process → download flow

## Phase 6: Progress Feedback (SSE)

- [ ] **T037** — Implement SSE endpoint `GET /api/progress/:jobId` — stream progress events as images are processed
- [ ] **T038** — Emit progress events from batch processing loop (current count, total, filename)
- [ ] **T039** — Emit complete event with download URL when processing finishes
- [ ] **T040** — Emit error event if processing fails
- [ ] **T041** — Client: connect to SSE after submitting, update progress display (e.g., "Processing 5/50...")
- [ ] **T042** — Client: transition to `complete` state on SSE complete event, show download button
- [ ] **T043** — Client: transition to `error` state on SSE error event, show error message
- [ ] **T044** — Write integration tests for SSE (progress events fire in order, complete event contains download URL)

## Phase 7: Cleanup & Error Handling

- [ ] **T045** — Implement temp file cleanup after download (delete job directory on response close)
- [ ] **T046** — Implement timeout-based cleanup (delete job directories older than 30 minutes)
- [ ] **T047** — Implement startup cleanup (clear any leftover temp directories)
- [ ] **T048** — Add Express error-handling middleware (catch-all, return JSON error responses)
- [ ] **T049** — Client: display user-friendly error messages for common failure scenarios (upload too large, invalid files, server error)
- [ ] **T050** — Write integration tests for cleanup (temp files are removed after download)

## Phase 8: Styling & Polish

- [ ] **T051** — Design CSS custom properties: color palette, spacing scale, typography
- [ ] **T052** — Style the upload area (drag-and-drop feel, file list)
- [ ] **T053** — Style the settings form (organized sections, clear labels)
- [ ] **T054** — Style the progress and completion views
- [ ] **T055** — Style the error state
- [ ] **T056** — Responsive layout (mobile-friendly)
- [ ] **T057** — Accessibility pass (labels, focus states, ARIA, keyboard navigation)

## Phase 9: End-to-End Tests

- [ ] **T058** — E2E: upload images, use default settings, download ZIP, verify contents
- [ ] **T059** — E2E: upload images, change settings (resize + format conversion), verify output
- [ ] **T060** — E2E: attempt upload with invalid files, verify error is shown
- [ ] **T061** — E2E: upload max number of files, verify progress feedback and successful download

## Phase 10: Preview (Phase 2 feature)

- [ ] **T062** — Implement `POST /api/preview` — accept single image + settings, return processed image
- [ ] **T063** — Client: show thumbnail of first uploaded image
- [ ] **T064** — Client: "Preview" button sends first image + current settings to preview endpoint, display result
- [ ] **T065** — Write tests for preview endpoint and UI

## Phase 11: Deployment

- [ ] **T066** — Create production start script and verify `npm start` works
- [ ] **T067** — Write `README.md` (project description, local setup, environment variables)
- [ ] **T068** — Deploy to Render (configure, connect to GitHub repo, verify)
- [ ] **T069** — Smoke test the deployed application

---

## Task Notes

- Tasks are ordered so that each builds on the previous ones.
- Some tasks within a phase can be done in parallel (e.g., T020–T027 are independent operations).
- The TDD workflow means the Test Agent may write test tasks (like T009, T013, T029) *before* the Implementer works on the corresponding feature tasks.
- If a task turns out to be too large for a single commit, the Planner Agent should split it.
