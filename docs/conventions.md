# Bubble — Coding Conventions

## General Principles

- **Readability over cleverness.** Code should be obvious to a newcomer.
- **Minimal abstractions.** Don't abstract until there's a clear repeated pattern.
- **Small files.** Each file should have a single, clear responsibility.
- **No dependencies on the frontend.** Zero npm packages in `client/`.
- **Minimal dependencies on the backend.** Only what's listed in the architecture doc.

## Language & Runtime

- **JavaScript** (no TypeScript) — keeps the toolchain simple.
- **ES modules** everywhere (`"type": "module"` in `package.json`).
- **Node.js 20 LTS** or later.
- **No transpilation, no bundlers, no build step** for frontend code. The browser runs the source files directly.
- Use modern browser APIs freely (no IE11 support needed).

## Naming

| Thing | Convention | Example |
|---|---|---|
| Files & folders | `kebab-case` | `image-processor.js`, `zip-builder.js` |
| Variables & functions | `camelCase` | `processedCount`, `buildZip()` |
| Constants | `UPPER_SNAKE_CASE` | `MAX_UPLOAD_SIZE`, `DEFAULT_QUALITY` |
| CSS classes | `kebab-case` | `.upload-area`, `.progress-bar` |
| CSS custom properties | `--kebab-case` | `--color-primary`, `--spacing-md` |
| HTML IDs | `camelCase` | `id="uploadForm"`, `id="progressCount"` |
| API routes | `lowercase` | `/api/process`, `/api/download/:jobId` |

## JavaScript Style

### Formatting

- **2-space indentation.**
- **Single quotes** for strings.
- **Semicolons** always.
- **Trailing commas** in multiline arrays, objects, and function parameters.
- **Max line length**: 100 characters (soft limit — don't break readability to hit it).
- **No `var`**. Use `const` by default, `let` when reassignment is needed.

### Functions

- Prefer **named function declarations** for top-level functions:
  ```js
  function processImage(file, settings) { ... }
  ```
- Use **arrow functions** for callbacks and inline functions:
  ```js
  images.map((img) => img.filename);
  ```
- One function, one job. If a function does two things, split it.

### Error Handling

- **Server**: Use try/catch and propagate errors to Express error middleware.
- **Client**: Use try/catch around fetch calls. Display user-friendly messages.
- Never silently swallow errors. At minimum, `console.error()` them.
- Validation errors should include a clear, human-readable message.

### Async

- Use `async/await` everywhere. No raw `.then()` chains.
- Process images sequentially (one at a time) to control memory usage. Do not `Promise.all()` the entire batch.

## Frontend Conventions

### HTML

- Semantic HTML elements where appropriate (`<main>`, `<section>`, `<form>`, `<button>`).
- All interactive elements must be accessible (labels for inputs, button text, ARIA where needed).
- Single `index.html` file. No routing, no SPA framework.

### CSS

- Single `styles.css` file (split into multiple files only if it grows past ~500 lines).
- Use **CSS custom properties** for colors, spacing, and other design tokens.
- Mobile-first responsive design using standard media queries.
- No CSS frameworks, no preprocessors.
- Use **flexbox** and **grid** for layout. No floats.

### JavaScript (Client)

- Each `.js` file in `client/js/` is an ES module.
- `app.js` is the entry point — imported via `<script type="module" src="js/app.js">`.
- **No global variables.** All communication between modules is through explicit imports/exports and the state machine.
- DOM queries use `document.getElementById()` or `document.querySelector()`. Cache DOM references at module load time, not on every function call.
- Event listeners are attached in `app.js` or `ui.js`, not scattered across files.

### State Machine

- `state-machine.js` contains the generic state machine implementation (reusable).
- `app-machine.js` defines the Bubble-specific states, transitions, and side effects.
- The state machine is the **single source of truth** for what the UI should display.
- UI updates happen in response to state transitions, never ad-hoc.

## Backend Conventions

### Express

- `app.js` configures Express (middleware, routes, error handling). Does not call `listen()`.
- `index.js` imports the app and calls `listen()`. This separation allows testing the app without starting the server.
- Routes are grouped in `server/routes/`. Each route file exports a router.
- Business logic lives in `server/services/`, not in route handlers.
- Route handlers are thin: validate input → call service → send response.

### File Organization

- `routes/` — Express route definitions
- `services/` — Business logic (image processing, ZIP creation)
- `middleware/` — Express middleware (upload handling)
- `utils/` — Pure utility functions (cleanup, filename sanitization)

### Environment

- Use `process.env` for configuration (port, temp directory, etc.).
- Provide sensible defaults. The app should work with zero environment variables in development.
- **No `.env` files in the repo.** Document required variables in README.

## Testing Conventions

### File Naming

- Unit tests: `tests/unit/<module-name>.test.js`
- Integration tests: `tests/integration/<feature>.test.js`
- E2E tests: `tests/e2e/<flow>.test.js`

### Test Style

- Each test file mirrors the module it tests.
- Use descriptive test names: `'returns 400 when no files are uploaded'`.
- Follow **Arrange → Act → Assert** structure.
- Tests must be independent — no shared mutable state between tests.
- No mocking unless absolutely necessary. Prefer testing real behavior.

### TDD Workflow

- The **Test Agent** writes failing tests before the Implementer writes code.
- Tests are committed separately from implementation.
- After implementation, the Test Agent may add additional edge case tests.

## Commit Messages

Format: `<type>: <short description>`

Types:
- `feat` — new feature
- `fix` — bug fix
- `test` — adding or updating tests
- `docs` — documentation changes
- `refactor` — code change that doesn't add a feature or fix a bug
- `chore` — tooling, config, dependencies

Examples:
- `feat: add image upload endpoint`
- `test: add unit tests for image-processor resize`
- `docs: update architecture with SSE details`

Keep the description under 72 characters. Use imperative mood ("add", not "added").

## Code Comments

- **Don't comment _what_ the code does** — the code should be readable enough.
- **Do comment _why_** when the reason isn't obvious.
- Use `// TODO:` for known missing pieces that will be addressed in a future task.
- Use `// HACK:` for intentional workarounds that should be revisited.
