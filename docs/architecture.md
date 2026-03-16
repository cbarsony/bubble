# Bubble — Architecture

## Tech Stack

| Layer | Technology | Notes |
|---|---|---|
| Frontend | Vanilla HTML, CSS, JavaScript | Zero dependencies. Modern browser APIs only. |
| Backend | Node.js 20+ (LTS), ES modules | |
| HTTP framework | Express | |
| Image processing | Sharp | |
| ZIP creation | Archiver | Streams processed images into a ZIP file |
| File uploads | Multer | Express middleware for multipart/form-data |
| Testing (unit/integration) | Vitest | |
| Testing (E2E) | Playwright | |

## Monorepo Structure

```
bubble/
├── client/                     # Frontend (served as static files)
│   ├── index.html
│   ├── css/
│   │   └── styles.css
│   └── js/
│       ├── app.js              # Entry point, wires everything together
│       ├── state-machine.js    # Generic state machine implementation
│       ├── app-machine.js      # App-specific states & transitions
│       ├── ui.js               # DOM manipulation, rendering
│       ├── api.js              # Server communication (fetch, SSE)
│       └── validation.js       # Client-side input validation
│
├── server/
│   ├── index.js                # Entry point, starts Express
│   ├── app.js                  # Express app setup (middleware, routes)
│   ├── routes/
│   │   └── process.js          # /api/process, /api/progress, /api/download
│   ├── services/
│   │   ├── image-processor.js  # Sharp-based image processing
│   │   └── zip-builder.js      # Archiver-based ZIP creation
│   ├── middleware/
│   │   └── upload.js           # Multer configuration
│   └── utils/
│       └── cleanup.js          # Temp file cleanup
│
├── tests/
│   ├── unit/                   # Unit tests (pure functions, services)
│   ├── integration/            # API-level tests (HTTP requests to server)
│   └── e2e/                    # Full browser tests (Playwright)
│
├── docs/
│   ├── requirements.md
│   ├── architecture.md
│   ├── conventions.md
│   ├── tasks.md
│   └── agents/                 # Per-agent folders (see agents.md)
│       ├── knowledge-agent/
│       ├── architect-agent/
│       ├── planner-agent/
│       ├── implementer-agent/
│       ├── test-agent/
│       └── reviewer-agent/
│
├── package.json
└── README.md
```

## API Design

### `POST /api/process`

Uploads images and processing settings. Returns a job ID immediately.

**Request**: `multipart/form-data`
- `images`: file field (multiple files)
- `settings`: JSON string with processing options

**Response** (202 Accepted):
```json
{
  "jobId": "abc123"
}
```

Processing begins asynchronously after the response is sent.

### `GET /api/progress/:jobId`

Server-Sent Events (SSE) stream. The client connects after receiving the job ID.

**Events**:
```
event: progress
data: {"current": 5, "total": 50, "filename": "photo.jpg"}

event: complete
data: {"downloadUrl": "/api/download/abc123"}

event: error
data: {"message": "Failed to process image: corrupt.jpg"}
```

### `GET /api/download/:jobId`

Returns the ZIP file. Available only after processing is complete.

**Response**: `application/zip` binary stream.

After the download completes, the server schedules cleanup of temporary files.

### `POST /api/preview` (Phase 2)

Sends the first uploaded image with current settings, returns the processed result.

**Request**: `multipart/form-data`
- `image`: single file
- `settings`: JSON string

**Response**: The processed image binary (with appropriate `Content-Type`).

## Processing Settings Schema

```json
{
  "resize": {
    "enabled": false,
    "mode": "percentage",
    "percentage": 100,
    "width": null,
    "height": null,
    "maintainAspectRatio": true
  },
  "crop": {
    "enabled": false,
    "aspectRatio": "1:1"
  },
  "rotate": {
    "enabled": false,
    "angle": 90
  },
  "autoRotate": {
    "enabled": true
  },
  "quality": {
    "enabled": false,
    "value": 80
  },
  "format": {
    "convert": false,
    "outputFormat": "jpeg"
  },
  "brightness": {
    "enabled": false,
    "value": 0
  },
  "contrast": {
    "enabled": false,
    "value": 0
  },
  "watermark": {
    "enabled": false,
    "text": "",
    "position": "bottom-right",
    "opacity": 0.5,
    "fontSize": 24
  },
  "stripMetadata": {
    "enabled": false
  }
}
```

## Request Lifecycle

```
1. User selects files + configures settings in the browser
2. Client sends POST /api/process (multipart: files + settings JSON)
3. Server:
   a. Generates a unique job ID
   b. Saves uploaded files to a temp directory: /tmp/bubble/<jobId>/input/
   c. Responds with 202 { jobId }
   d. Begins processing in the background
4. Client connects to GET /api/progress/<jobId> (SSE)
5. Server processes each image:
   a. Reads from input dir
   b. Applies operations via Sharp (in the order defined below)
   c. Writes to /tmp/bubble/<jobId>/output/
   d. Sends SSE progress event after each image
6. Server creates ZIP from output directory
7. Server sends SSE complete event with download URL
8. Client triggers download of GET /api/download/<jobId>
9. After download completes (or after a timeout), server deletes /tmp/bubble/<jobId>/
```

## Operation Order

When multiple operations are enabled, they are applied in this fixed order:

1. Auto-rotate (EXIF)
2. Rotate (manual angle)
3. Crop
4. Resize
5. Brightness
6. Contrast
7. Watermark
8. Strip metadata
9. Format conversion / quality

This order is intentional:
- Rotation before crop/resize so dimensions are correct.
- Crop before resize so you crop the original composition, then scale.
- Watermark after all geometric operations so it's not distorted.
- Metadata stripping and format conversion last since they don't affect pixel data.

## Frontend State Machine

The UI is orchestrated by a simple statechart. States and transitions:

```
┌─────────┐   files selected    ┌────────────┐
│  idle    │ ──────────────────► │  selected  │
└─────────┘                     └────────────┘
     ▲                               │
     │        files cleared          │  submit
     └───────────────────────────────│──────────────┐
                                     ▼              │
                                ┌────────────┐      │
                                │ processing │      │
                                └────────────┘      │
                                     │              │
                              ┌──────┴──────┐       │
                              ▼             ▼       │
                         ┌─────────┐  ┌─────────┐  │
                         │ complete│  │  error   │  │
                         └─────────┘  └─────────┘  │
                              │             │       │
                              └─────────────┘       │
                                     │ reset        │
                                     ▼              │
                                ┌─────────┐         │
                                │  idle    │ ◄──────┘
                                └─────────┘
```

**States**:
- `idle` — No files selected. Form is empty.
- `selected` — Files chosen. User configures settings. Submit button enabled.
- `processing` — Batch is being processed. Form is disabled. Progress is shown.
- `complete` — Processing done. Download button shown.
- `error` — Something went wrong. Error message shown. User can retry or reset.

**The state machine controls which UI elements are visible/enabled.** DOM manipulation is done by the `ui.js` module reacting to state changes.

## Temporary File Management

- All temp files live under a system temp directory (e.g., `os.tmpdir()/bubble/<jobId>/`).
- Cleanup happens:
  - After the ZIP is downloaded (response `close` event)
  - On a timeout (e.g., 30 minutes after job creation) as a safety net
  - On server startup (clear any leftover temp directories)

## Deployment

Target: a simple platform-as-a-service that requires minimal configuration.

Recommended options (in order of simplicity):
1. **Render** — free tier available, auto-deploys from GitHub, zero config
2. **Railway** — similar to Render, slightly more flexible
3. **Fly.io** — good for more control, but slightly more setup

The server serves the `client/` directory as static files, so there is only one deployable unit (the Node.js server).

A single `npm start` command starts everything.

## Security Considerations (MVP)

- **File type validation**: Check MIME type and file extension on upload. Reject non-image files.
- **File size limits**: Enforced by Multer configuration.
- **Temp directory isolation**: Each job gets its own directory. Job IDs are UUIDs.
- **No directory traversal**: Filenames are sanitized before writing to disk.
- **Cleanup**: Aggressive temp file cleanup prevents disk exhaustion.
- **No auth**: Public access. Rate limiting is a future concern.
