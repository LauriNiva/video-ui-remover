# AGENTS.md

## Project intent

Build a **small, focused macOS utility** that removes one fixed BG3 UI element from gameplay video.

The project should remain simple unless real usage proves more complexity is needed.

## Read first

Before making changes, read:

1. `PROJECT_STATE.md`
2. `TODO_SIMPLE.md`
3. `TODO.md`
4. `USER_ACTIONS.md`

Update these files when implementation materially changes project state.

## Core constraints

- Primary platform: macOS on Apple Silicon.
- Primary test machine: M4 MacBook Air, 16 GB unified memory.
- Primary footage: 3840×2160 BG3 PS5 video.
- UI to remove: fixed "Move To" overlay near the center of the screen.
- The scenery moves; the UI coordinates do not.
- Use local video inpainting.
- Prefer ProPainter / propainter-delogo rather than implementing ML from scratch.
- Prefer processing only a small padded crop around the UI.
- Desktop stack: Electron + React + TypeScript.
- Keep all heavy video/ML work outside the Electron renderer.
- Use the Electron main process to launch the local backend.
- Keep preload IPC narrow and typed.
- Do not introduce an LLM dependency into the shipped app.
- Never overwrite the source video.

## Avoid over-engineering

Do not add without a concrete need:

- database
- backend server
- account system
- cloud service
- plugin architecture
- generic workflow engine
- arbitrary video editor features
- automatic object detection
- segmentation model
- OCR
- telemetry
- elaborate design system
- premature abstractions for multiple games

Prefer straightforward TypeScript and explicit IPC contracts over frameworks or generic abstractions.

## Electron safety / architecture

- Do not enable Node integration in the renderer.
- Use context isolation.
- Expose only required operations through preload.
- Do not send raw video buffers over IPC.
- Do not execute user paths through shell strings.
- Use argument arrays with spawned processes.
- Treat the Python/ProPainter backend as a separate local process.

## Development order

1. Prove the inpainting result manually.
2. Make the backend invocation reproducible.
3. Build the smallest Electron wrapper.
4. Add error handling.
5. Improve UX only after the core result is good.

Do not spend significant time polishing the GUI before Phase 0 succeeds.

## UI principles

The normal user should only need to understand:

- choose/drop video
- Normal vs High quality
- Clean Video
- processing state
- where the output was saved

Backend concepts such as FFmpeg, MPS, RAFT, PyTorch, crop scale, masks, and model weights should not appear in the normal UI unless needed to explain an error.

## Planning file maintenance

Update `PROJECT_STATE.md` when architecture, backend/model choice, scope, or validated assumptions change.

Keep `TODO_SIMPLE.md` short and human-readable.

Use `TODO.md` for detailed implementation steps.

Add entries to `USER_ACTIONS.md` only when a human must answer a question, perform a manual test, provide footage, or change something outside the repo.

## Agent behavior

When blocked by a missing user action:

1. record it in `USER_ACTIONS.md`
2. continue with independent work that is still safe and useful
3. do not invent test results

When completing meaningful work:

1. update the relevant TODO files
2. update PROJECT_STATE if assumptions changed
3. note required manual verification in USER_ACTIONS
