# Depth Map Generator

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/saintdaniel009-cell/depth-map-generator/blob/main/Depth_Map_Generator.ipynb)

A Colab notebook that turns video clips into grayscale depth maps (bright = near, dark = far) for depth-based background blur in DaVinci Resolve. It pairs with my `Rigo_DepthBlur` Fusion macro, which reads the depth map and blurs the background of the original shot.

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

## Credits

Depth estimation by [Video Depth Anything](https://github.com/DepthAnything/Video-Depth-Anything) (ByteDance). The model weights are under their own licences; check them before any commercial use.
