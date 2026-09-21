# Project State

## Goal

Build a very simple macOS app that removes the fixed BG3 **"Move To"** UI element from gameplay video.

Primary target:

- Apple Silicon Mac
- initially tested on M4 MacBook Air with 16 GB unified memory
- mostly 3840×2160 (4K) PS5 footage
- 16:9 video
- moving game scenery behind a UI element that stays at fixed screen coordinates

## MVP

The app should:

1. Accept one video via drag-and-drop or file picker.
2. Inspect its resolution, frame rate, and duration.
3. Apply a fixed mask for the BG3 "Move To" UI.
4. Inpaint only a padded crop around that mask.
5. Composite the repaired crop back into the original frames.
6. Preserve audio.
7. Write a new output file without overwriting the source.
8. Show simple processing state.
9. Offer "Show in Finder" when complete.

## Technology decisions

### macOS UI

Use **Swift + SwiftUI**.

Keep the native app thin. It should mainly handle:

- file selection / drag-and-drop
- metadata display
- launching the backend process
- status / errors
- output handling

### Inpainting backend

Use **ProPainter via propainter-delogo** or an equivalent local wrapper that:

- can use Apple Silicon MPS
- processes only a cropped window around the unwanted UI
- composites the crop back into the source video
- preserves audio

Do not implement video inpainting from scratch.

### Video tools

Use FFmpeg / ffprobe for:

- probing video metadata
- video decoding / encoding where required
- audio preservation / remuxing

### Mask

The "Move To" UI stays at the same coordinates, so MVP does **not** need:

- object tracking
- OCR
- UI detection
- segmentation
- an LLM
- a moving mask

Store the mask using normalized coordinates so the same preset can scale to different 16:9 resolutions.

The production mask must be calibrated from native 3840×2160 footage.

## Initial UX

Single-window utility:

```
Video UI Remover

[ Drop video here ]
or
[ Choose Video ]

Input: gameplay.mp4
3840 × 2160 · 60 fps · 00:27

Quality:
(*) Normal
( ) High

[ Clean Video ]

Status / progress

Output: gameplay-clean.mp4
[ Show in Finder ]
```

## Processing modes

### Normal

Default for 4K.

Target configuration:

- cropped processing window
- `--proc-scale 0.5`
- reduced RAFT iterations where quality remains acceptable

### High

Fallback for difficult scenes.

Target configuration:

- cropped processing window
- `--proc-scale 1.0`
- normal/default optical-flow settings

Exact flags should be validated against the selected backend version instead of blindly hard-coding assumptions.

## Output rules

For:

`gameplay.mp4`

write:

`gameplay-clean.mp4`

If it exists:

`gameplay-clean-2.mp4`

Never overwrite the source file.

## First-run / packaging strategy

For MVP, do **not** spend time creating a fully self-contained distributable app.

Use a one-time local backend setup, for example:

```
~/Library/Application Support/VideoUIRemover/
├── venv/
├── propainter-delogo/
└── model-weights/
```

The SwiftUI app may assume this backend is installed.

A setup script can install/check:

- Python environment
- FFmpeg
- PyTorch
- required Python packages
- ProPainter / wrapper
- model weights

Later, if the app proves useful, package dependencies more cleanly.

## Non-goals for MVP

Do not add yet:

- video timeline editing
- batch queue
- cloud/API processing
- user accounts
- automatic UI detection
- minimap removal
- party HUD removal
- arbitrary masks
- multiple games
- automatic App Store distribution
- bundled Python/PyTorch runtime
- LLM integration

## Validation target

The first milestone is successful cleanup of a short native 4K BG3 clip where:

- the "Move To" element is visible
- the camera/scenery moves behind it
- output does not visibly flicker or leave a distracting patch
- audio remains intact
