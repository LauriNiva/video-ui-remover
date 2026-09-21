# TODO

## Phase 0 — Verify the core technique

The UI wrapper should not be built until the cleanup result is proven good enough.

### Tasks

- Obtain a native 3840×2160 BG3 PS5 recording with the "Move To" UI visible.
- Install/test the selected ProPainter-based backend on the M4 MacBook Air.
- Verify Apple Silicon MPS is actually used where expected.
- Define a mask that fully covers:
  - center circle
  - X-button glyph
  - "Move To" text
  - small safety margin
- Process a 3–10 second clip.
- Inspect static backgrounds, foliage, water, moving characters, camera pans, lighting, and particle effects.
- Confirm audio survives the output pipeline.
- Compare Normal vs High settings.
- Record approximate processing speed and peak memory.

### Exit criteria

Proceed when a representative test clip looks good enough that the removed UI is less noticeable than the original overlay.

---

## Phase 1 — Backend setup

Create a reproducible local processing environment.

### Proposed location

```
~/Library/Application Support/VideoUIRemover/
```

### Add

- Python virtual environment
- FFmpeg / ffprobe check
- ProPainter-delogo or selected wrapper
- ProPainter model files
- dependency installation
- setup/check script

### Requirements

- fail clearly if a dependency is missing
- avoid modifying source video
- support paths containing spaces
- return useful exit codes and stderr
- expose a simple CLI contract for Electron

Example target interface:

```bash
video-ui-remover-backend \
  --input "/path/input.mp4" \
  --output "/path/input-clean.mp4" \
  --preset bg3-move-to \
  --quality normal
```

---

## Phase 2 — Preset / mask model

Create a simple preset model using normalized coordinates.

Conceptual structure:

```
BG3MoveToPreset
- centerX
- centerY
- width
- height
- padding
```

MVP support:

- 16:9 footage
- primary target: 3840×2160
- optionally scale automatically to 1920×1080 and 2560×1440

Reject unsupported layouts clearly rather than guessing.

---

## Phase 3 — Electron shell

Create the desktop app with:

- Electron
- React
- TypeScript

Suggested structure:

```
app/
├── main/
│   ├── main.ts
│   ├── ipc.ts
│   ├── processRunner.ts
│   └── videoProbe.ts
├── preload/
│   └── preload.ts
└── renderer/
    ├── App.tsx
    ├── components/
    └── types/
```

Architecture rules:

- renderer must not process video
- renderer must not receive raw video buffers
- preload exposes a minimal typed API
- main process owns filesystem access, native dialogs, Finder integration, and backend process execution
- backend runs as a separate child process
- IPC should carry only paths, commands, progress, status, and errors

### Main screen

Support:

- drag-and-drop one file
- Choose Video button
- input filename
- resolution
- FPS
- duration
- Normal / High toggle
- Clean Video
- processing status
- completion state
- Show in Finder

Keep backend details hidden.

---

## Phase 4 — Video metadata

Use ffprobe from the main/backend layer.

Read at minimum:

- width
- height
- frame rate
- duration
- video codec
- presence of audio

Validate supported container, 16:9 layout, sensible resolution, and readable input.

---

## Phase 5 — Process execution

Use Node's `child_process.spawn()` in the Electron main process.

Responsibilities:

- launch backend executable directly
- pass arguments as an argument array
- capture stdout/stderr
- surface failures through typed IPC
- support cancellation if straightforward
- avoid shell interpolation for user-controlled paths
- keep processing fully outside the renderer

---

## Phase 6 — Output handling

Generate output beside the source by default:

```
clip.mp4
→ clip-clean.mp4
→ clip-clean-2.mp4
→ clip-clean-3.mp4
```

Never overwrite.

After success:

- show output path
- provide Show in Finder

---

## Phase 7 — Progress UX

Start with coarse states:

1. Preparing
2. Reading video
3. Cleaning UI
4. Encoding output
5. Finished

If backend output later provides reliable frame/scene progress, map it to a percentage.

---

## Phase 8 — Preview mode

Add later if useful:

- process a representative 3-second segment
- reveal or open the preview
- use it to verify mask and quality before a long run

---

## Phase 9 — Developer mask calibration

Optional debug/developer tool:

- extract one frame
- render it in the React UI
- overlay a draggable/resizable rectangle
- save normalized coordinates

Not part of the normal user flow.

---

## Phase 10 — Packaging and hardening

Test:

- filenames with spaces
- long paths
- missing backend
- missing weights
- failed FFmpeg
- out-of-memory conditions
- cancellation
- source without audio
- HEVC/H.264 input
- 30/60 fps
- long 4K clips
- sleep/wake during processing

For the first personal-use version, keep the backend externally installed.

Only later investigate bundling:

- Electron packaging
- signing/notarization
- FFmpeg distribution
- Python/runtime bundling
- model weight download/setup

---

## Later ideas

Only consider after the MVP is useful:

- batch cleanup
- queue
- additional BG3 HUD presets
- arbitrary user-drawn masks
- before/after comparison
- completion notifications
- bundled runtime
- updater
- Windows support
- other games
- optional cloud backend
