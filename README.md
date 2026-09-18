# Local AI YouTube Video Automation Pipeline

Local-first React + Vite + Express + TypeScript + FFmpeg application for turning a topic into a script, AI-generated visuals, Gemini TTS narration, captions, and an upload-ready MP4.

## Important fixes in this version
- Long-running generation is now a persisted **PipelineJob** instead of a long HTTP request.
- Real-time updates use **Server-Sent Events** at `/api/jobs/:id/events`.
- Jobs can be cancelled and retried; completed assets are reused when a job is restarted.
- Project IDs and asset paths are validated to prevent path traversal.
- Project PATCH accepts an explicit allow-list instead of `Object.assign(req.body)`.
- Project JSON writes are atomic and old projects are normalized when loaded.
- AI failures are real failures: there are **no fake image placeholders, fake TTS tones, or false success states**.
- Gemini models are configurable through environment variables rather than scattered hard-coded names.
- FFmpeg uses argument arrays instead of shell command strings.
- Actual measured TTS durations are used for rendering and captions.
- Storage now reserves folders for scenes, audio, music, SFX, captions, thumbnails, renders, temp files, and versions.

## Gemini configuration
The current Google documentation lists Gemini 2.5 Flash for text, Gemini 3.1 Flash Image for image generation, and Gemini 3.1 Flash TTS Preview for speech generation. The project therefore uses these as configurable defaults; change them in `.env` if your account/project requires another supported model.

Create `.env`:

```env
GEMINI_API_KEY=your_key
GEMINI_TEXT_MODEL=gemini-2.5-flash
GEMINI_IMAGE_MODEL=gemini-3.1-flash-image
GEMINI_TTS_MODEL=gemini-3.1-flash-tts-preview
PORT=3000
FFMPEG_PATH=ffmpeg
FFPROBE_PATH=ffprobe
FFMPEG_PRESET=medium
```

Google's current image-generation documentation recommends the newer Nano Banana image models and notes that Imagen models are deprecated, so this project no longer uses Imagen as a fallback.

## Run

Requirements:
- Node.js 20+
- FFmpeg + ffprobe on PATH, or set `FFMPEG_PATH` / `FFPROBE_PATH`
- A Google AI Studio API key with access to the configured models

```bash
npm install
npm run dev
```

Open `http://localhost:3000`.

## API

- `POST /api/projects` — create project
- `GET /api/projects` — list projects
- `GET /api/projects/:id` — project
- `PATCH /api/projects/:id` — validated project edits
- `POST /api/projects/:id/generate-script`
- `POST /api/projects/:id/generate-images`
- `POST /api/projects/:id/generate-voiceover`
- `POST /api/projects/:id/scenes/:sceneIndex/regenerate-image`
- `POST /api/projects/:id/scenes/:sceneIndex/regenerate-audio`
- `POST /api/projects/:id/run-pipeline` — returns HTTP 202 + job immediately
- `GET /api/jobs/:id` — job status
- `GET /api/jobs/:id/events` — SSE event stream
- `POST /api/jobs/:id/cancel`
- `POST /api/jobs/:id/retry`

## Pipeline

Create Project → Script → Visuals → Voiceover → Render → Captions → Thumbnail → QA → Export.

The pipeline intentionally fails instead of silently replacing failed AI assets. A user can retry the job or regenerate an individual asset.

## Validation and testing

The source was syntax-checked with the installed TypeScript compiler. Full dependency installation/build could not be completed in this environment because `npm install` timed out, so the final package should be run with `npm install`, `npm run lint`, and `npm run build` on the target machine before production use.

## Storage

```text
storage/
  projects/<project-id>/
    project.json
    scenes/
    audio/
    music/
    sfx/
    captions/
    thumbnails/
    renders/
    temp/
    versions/
  jobs/
    <job-id>.json
    <job-id>.events.ndjson
```

Never put API keys into project JSON or exported project files.
