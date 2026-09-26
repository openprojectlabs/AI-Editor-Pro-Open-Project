# AI Editor Pro

AI Editor Pro is a local-first, AI-assisted non-linear video editor. The restored project keeps the real editor implementation in the repository-level `src/` tree and exposes portable `frontend/` and `backend/` workspace launchers so the application can be run from any checkout directory.

## Requirements

- Node.js 24.x recommended. Node 22.18+ is supported.
- npm 10+
- FFmpeg/ffprobe are bundled through npm where the platform package is available; system FFmpeg/ffprobe can be supplied with `FFMPEG_PATH` and `FFPROBE_PATH`.
- Optional: a supported GPU/driver for hardware acceleration.

## Structure

```text
AI-Editor-Pro/
├── package.json
├── package-lock.json
├── .env.example
├── frontend/
│   ├── package.json
│   ├── src/main.tsx
│   └── public/
├── backend/
│   ├── package.json
│   ├── src/server.ts
│   └── Dockerfile
├── src/                 # React editor and timeline implementation
├── server/              # API, media, AI, export and job services
├── shared/              # Shared contracts and provider models
├── public/              # Runtime media/model assets
├── assets/              # Static editor assets/templates
├── config/              # Vite/build configuration
├── scripts/             # Portable launch, test, E2E and diagnostics scripts
├── remotion/            # Preview/render composition
└── desktop/             # Optional Electron shell
```

No `.lnk` shortcut is required. No script depends on a particular Windows username or absolute checkout path.

## Install

```bash
npm install
npm run install:all
```

`npm run install:all` installs the root package and both workspace launchers. Run it from the project root.

## Development

Start the complete application:

```bash
npm run dev
```

Or start each service separately:

```bash
npm run dev:backend
npm run dev:frontend
```

Default development endpoints:

- Frontend: http://127.0.0.1:5173
- Backend: http://127.0.0.1:5180
- Health: http://127.0.0.1:5180/api/health
- Diagnostics: http://127.0.0.1:5180/api/diagnostics

The frontend proxies `/api` requests to the backend.

## Build and quality checks

```bash
npm run lint
npm run typecheck
npm test
npm run build
```

For a real local media smoke test, with FFmpeg available:

```bash
npm run test:e2e
```

The E2E flow starts the backend, checks health and FFmpeg/FFprobe, creates a real MP4 fixture, uploads it through `/upload`, probes the uploaded media, builds a real timeline state, renders it through `/render-clip`, and verifies that a non-empty output file is produced.

Diagnostics from a running backend:

```bash
npm run diagnostics
```

## AI providers and API keys

Copy `.env.example` to `.env.local` and configure only the providers you use. Provider keys are server-side. Do not use `VITE_*` variables for secrets.

The editor's Settings UI can also store supported provider credentials through the backend key store. The API returns configuration status/model information rather than secret values.

The AI architecture is provider-based: speech-to-text, video analysis, caption generation, editing suggestions and text generation can select independent providers where supported. Local/offline features are kept separate from external provider calls.

## Media processing

The editor uses editable timeline state rather than flattening every edit into a new master file. Source media is retained and edits store trims, transforms, effects, captions, audio changes and other timeline operations. Server rendering materializes the required local media only when an export/render is requested.

Supported workflows include uploads, metadata probing, scene detection, highlight detection, captions/transcription, reframing, effects/transitions, audio processing, project persistence, version history and export.

Hardware encoding/decoding is probed at runtime and falls back to software encoding when a compatible accelerator is not available.

## Security

- Uploaded filenames are sanitized.
- Upload limits are enforced server-side.
- Uploaded files are treated as data and are never executed.
- Temporary files are created in controlled directories and cleaned after operations.
- Project/media paths are validated to prevent path traversal.
- API keys remain on the server side.
- Backend errors are scrubbed before internal paths/secrets can be returned to clients.

## Windows

Open PowerShell in the folder containing this README and run the same npm commands above. The deployment and Git helpers derive their root from `$PSScriptRoot`; they do not assume `C:\Users\<username>` or any other username.

## Linux/macOS

Use the same npm commands. If the bundled FFmpeg binary is not usable on the platform, install FFmpeg with the OS package manager and set `FFMPEG_PATH` / `FFPROBE_PATH` if needed.

## Troubleshooting

1. Check Node: `node --version` (22.18+; 24.x recommended).
2. Run `npm install` again if `node_modules` is incomplete.
3. Run `npm run diagnostics` while the backend is running.
4. Check `/api/health` for backend availability.
5. Check FFmpeg/FFprobe status in the Diagnostics panel from the project dashboard.
6. Check provider configuration in Settings. Invalid or unavailable providers are reported instead of silently falling back to fake output.
7. For large projects, enable proxy/low-resolution preview options where available and use background export processing.

## Environment variables

See `.env.example` and `backend/.env.example`. Common local overrides include:

- `AI_EDITOR_BACKEND_HOST`
- `AI_EDITOR_BACKEND_PORT`
- `AI_EDITOR_FRONTEND_ORIGIN`
- `AI_EDITOR_BACKEND_URL`
- `AI_EDITOR_DATA_DIR`
- `FFMPEG_PATH`
- `FFPROBE_PATH`
- provider API keys and model/base-URL settings

Never commit `.env.local`, `.env.production`, or files containing credentials.
