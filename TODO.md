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
- Inspect:
  - static backgrounds
  - grass / foliage
  - water
  - moving characters
  - camera pans
  - lighting / particle effects
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
- expose a simple CLI contract for the GUI

Example target interface:

```bash
video-ui-remover-backend \
  --input "/path/input.mp4" \
  --output "/path/input-clean.mp4" \
  --preset bg3-move-to \
  --quality normal
```

The exact implementation may call propainter-delogo underneath.

---

## Phase 2 — Preset / mask model

Create a simple preset model.

Store normalized values rather than only absolute 4K pixels.

Example conceptual structure:

```
BG3MoveToPreset
- centerX
- centerY
- width
- height
- padding
```

Convert to integer pixel coordinates after probing the video.

### MVP support

- 16:9 footage
- primary target: 3840×2160
- optionally allow 1920×1080 and 2560×1440 automatically through normalized scaling

Reject unsupported layouts clearly rather than guessing.

---

## Phase 3 — SwiftUI shell

Create the macOS app.

Suggested structure:

```
VideoUIRemover/
├── App/
├── Models/
│   ├── VideoInfo.swift
│   └── CleanupPreset.swift
├── Services/
│   ├── VideoProbe.swift
│   ├── ProcessRunner.swift
│   └── CleanupService.swift
└── Views/
    ├── ContentView.swift
    ├── DropZone.swift
    └── ProcessingStatusView.swift
```

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

Keep all advanced/backend details hidden.

---

## Phase 4 — Video metadata

Use ffprobe.

Read at minimum:

- width
- height
- frame rate
- duration
- video codec
- presence of audio

Validate:

- supported extension/container
- 16:9
- sensible resolution
- readable file

Do not require a specific codec if the backend can decode it.

---

## Phase 5 — Process execution

Implement a small ProcessRunner around Foundation `Process`.

Responsibilities:

- launch backend executable
- pass arguments safely
- capture stdout/stderr
- surface failures
- support cancellation if straightforward
- never invoke a shell just to interpolate user-controlled paths

CleanupService should translate app-level options into backend arguments.

---

## Phase 6 — Output handling

Generate output alongside the source by default.

Rules:

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
- optionally provide Open With… later

---

## Phase 7 — Progress UX

Start simple.

Acceptable MVP states:

1. Preparing
2. Reading video
3. Cleaning UI
4. Encoding output
5. Finished

If backend stdout provides reliable frame/scene progress, map it to a percentage later.

Avoid pretending to have exact progress if it is not reliable.

---

## Phase 8 — Preview mode

Useful but not required before the first end-to-end app.

Add a "Test 3 seconds" action that:

- chooses a representative timestamp
- extracts/processes a short section
- opens or reveals the result

This is useful for verifying the mask and quality before processing a long clip.

---

## Phase 9 — Developer mask calibration

Add only after the main path works.

A simple internal/debug tool can:

- extract one frame
- display it
- overlay a draggable/resizable rectangle
- save normalized coordinates

This is for tuning presets, not for normal users.

---

## Phase 10 — Hardening

Test:

- filenames with spaces
- long paths
- missing backend
- missing weights
- failed FFmpeg
- out-of-memory conditions
- cancelled job
- source without audio
- HEVC/H.264 input
- 30/60 fps footage
- long 4K clips
- sleep/wake during processing

---

## Later ideas

Only consider after the MVP is useful:

- batch cleanup
- multiple input files
- queue
- additional BG3 HUD presets
- arbitrary user-drawn masks
- before/after comparison
- background processing notifications
- bundled runtime
- updater
- signed/notarized releases
- other games
- optional cloud backend
