# Depth Map Generator

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/saintdaniel009-cell/depth-map-generator/blob/main/Depth_Map_Generator.ipynb)

A Colab notebook that turns video clips into grayscale depth maps (bright = near, dark = far) for depth-based background blur in DaVinci Resolve. It pairs with `Rigo_DepthBlur`, a Fusion macro (included) that reads the depth map and blurs the background of the original shot.

It runs [Video Depth Anything](https://github.com/DepthAnything/Video-Depth-Anything), which keeps depth consistent from frame to frame, so the blur doesn't flicker.

## How to use

1. Open the notebook in Colab (badge above). For speed, set **Runtime → Change runtime type → T4 GPU**.
2. Put clips in **Google Drive → `DepthMaps/in`**. The folder is created on the first run.
3. Run **Cell 1** (setup, about 2 minutes), then **Cell 2**.
4. Collect `<clipname>_depth.mp4` from `DepthMaps/out`. It has the same length and frame rate as the source. Processed clips move to `DepthMaps/in/done`.

No GPU? Use the CPU runtime and run **Cell 2b** instead. It uses the small model, a lower resolution, and can compute every second frame. Its `test_frames` setting times a short run first so you know how long the full clip will take.

## What it handles

- **Long clips without splitting.** It estimates memory use from the clip's length and resolution, then picks the highest resolution that fits. That avoids the brightness seams you get when stitching chunks together.
- **Out-of-memory crashes.** It patches the model's attention to use PyTorch's memory-efficient attention, so the large model runs on a free T4 without xFormers.
- **Awkward source files.** iPhone `.mov` and other variable-frame-rate files are re-encoded to constant-frame-rate H.264 before processing.
- **Re-runs.** Model weights are cached in Drive and only download once.

## The Resolve macro: `Rigo_DepthBlur`

`Rigo_DepthBlur.setting` is a Fusion macro that uses the depth map to blur the original footage.

**Install:** copy it to `~/Library/Application Support/Blackmagic Design/DaVinci Resolve/Fusion/Macros/` (Windows: `%AppData%\Blackmagic Design\DaVinci Resolve\Support\Fusion\Macros\`) and restart Resolve. On the Fusion page it appears under **Add Tool → Macros**.

**Wire it up:** connect your clip to **Footage** and the generated `_depth.mp4` to **Depth Map**. Line the two up on the timeline so their frames match.

**Controls**
- **Focus Point / Focus Range:** the depth band that stays sharp (0 = far, 1 = near).
- **Falloff:** how gradually things outside that band go soft.
- **Background Blur / Foreground Blur:** how much to blur behind and in front of the focus band.
- **Blur Size / Edge Softness:** blur strength, and how soft the matte edges are.
- **Invert Depth:** for depth maps where dark means near.
- **Extra Blur Mask:** paint in extra blur where the depth map gets it wrong.
- **Show Blur Matte:** shows the matte so you can check what will be blurred.

**How it works:** a custom tool turns depth into a blur matte from the focus settings. The matte is split in two: the transition zone gets a lighter blur (40% strength) and the far zone gets the full blur. Layering the two gives a gradual fall-off instead of a hard edge between sharp and blurred.

## Credits

Depth estimation by [Video Depth Anything](https://github.com/DepthAnything/Video-Depth-Anything) (ByteDance). The model weights are under their own licences; check them before any commercial use.
