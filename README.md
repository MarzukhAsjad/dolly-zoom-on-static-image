# Dolly Zoom on a Static Image

This repository contains a Google Colab-style notebook that creates a **dolly zoom / “shrunken world” effect** from a single still image. The notebook keeps the detected subject visually stable while the background is scaled around that subject, then exports the result as an animated GIF.

## Notebook

- Main notebook: `dolly_zoom_on_static_image.ipynb`
- Colab link: https://colab.research.google.com/drive/1n_fnndhpB3-hNfewv_giLyfnigqnq3NT?usp=sharing

## How it works

The notebook builds up the effect in stages:

1. **Load and inspect an image**
   - The early cells load an image with OpenCV and display it in Colab.

2. **Prototype subject localization**
   - Initial cells use a simple color-based mask to find a red object and test the idea of preserving a region while scaling the rest of the frame.

3. **Detect the main subject**
   - A pre-trained SSD MobileNet model is downloaded and used to detect a `person` bounding box in the target image.

4. **Create a cleaner foreground mask**
   - A pre-trained Mask R-CNN model is used to segment the person more accurately than a plain rectangle.
   - The mask is saved and reused for later compositing.

5. **Scale the background around the subject**
   - The subject center is computed from the segmentation mask.
   - The full image is warped with a scale transform around that center so the world appears to shrink away from the subject.

6. **Composite the original subject over the transformed background**
   - The original person pixels are blended back onto the scaled background using the segmentation mask.
   - This preserves the subject while the environment changes around it.

7. **Crop to the visible transformed region**
   - The transformed image bounds are estimated and clipped so every animation frame uses a consistent crop.

8. **Generate animation frames**
   - Multiple scale values are interpolated from `1.0` to `0.7`.
   - For each scale, the background is transformed, the subject is composited back in, and the cropped frame is stored.

9. **Export a GIF**
   - The generated frames are converted to RGB and saved as `shrunken_world.gif`.

## Running it

This project is currently designed primarily for **Google Colab**.

1. Open the notebook from the Colab link above.
2. Upload or replace the source image in Colab storage.
3. Update any hardcoded `/content/...` image paths in the notebook cells if your filename differs.
4. Run the cells in order.
5. Wait for the notebook to:
   - install dependencies,
   - download the detection models,
   - detect and segment the subject,
   - generate animation frames,
   - save the final GIF.

## Current limitations

- The workflow is notebook-driven and manual.
- The image paths are hardcoded for Colab paths such as `/content/...`.
- The animation uses simple linear scaling, so motion can feel mechanical.
- Foreground edges depend on the quality of the segmentation mask.
- The pipeline is tuned mainly for a single detected person.

## Suggested improvements

### 1. Build an automated pipeline

- Convert the notebook into a reusable script or small app.
- Accept image input, output path, frame count, zoom strength, and FPS as parameters.
- Package model download/setup so the full effect can run in one command.
- Add automatic export of the final GIF or video.

### 2. Make the zoom depth-aware

- Apply stronger background motion to distant regions and less motion to near regions.
- Use monocular depth estimation or layered segmentation to separate foreground, midground, and background.
- This would make distant areas zoom further than nearer pixels and improve the cinematic feel.

### 3. Make the animation less robotic

- Replace strict linear interpolation with eased motion curves.
- Add subtle handheld-style jitter, micro drift, or natural camera sway.
- Introduce slight per-frame variation so the motion feels more organic and less synthetic.

### 4. Sharpen subject boundaries

- Refine the segmentation mask with edge-aware post-processing.
- Feather or clean mask borders to reduce halos and cutout artifacts.
- Consider higher-quality matting or segmentation models for hair, limbs, and thin edges.

### 5. Improve subject handling

- Support multiple subjects and automatic selection of the primary one.
- Handle cases where the detector misses the subject or the mask is incomplete.
- Allow manual correction of the detected center or mask when needed.

### 6. Expand output options

- Export MP4 in addition to GIF for better quality and smaller file sizes.
- Add controls for duration, scale range, crop strategy, and background fill.
- Save intermediate outputs for debugging: bounding box, mask, composite, and preview frames.

## Validation status

This repository currently contains a single notebook and no existing automated lint, build, or test commands were present to run for this documentation update.
