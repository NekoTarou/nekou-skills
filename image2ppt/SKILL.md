---
name: image2ppt
description: Convert images, screenshots, diagrams, flowcharts, architecture charts, and infographics into PowerPoint/PPTX decks, especially editable reconstructions using python-pptx and the bundled image2ppt.py scaffold. Use when the user says 图片转PPT, 图片转PowerPoint, 截图转PPT, convert image to PPT/PPTX, recreate an image as editable slides, or asks to use image2ppt.py.
---

# Image2PPT

## Overview

Use this skill to turn a source image into a `.pptx`. Prefer a fast static-image slide when the user only needs the image inside PowerPoint; recreate shapes, text, connectors, and layout with `python-pptx` when the user asks for an editable deck.

The bundled `scripts/image2ppt.py` is a known working scaffold from the user's workflow. Copy it into the working directory and rewrite `build_deck()` for the current image rather than starting from an empty script.

## Workflow

1. Determine the target:
   - Static PPT: place each image on one slide with preserved aspect ratio.
   - Editable PPT: rebuild the visual using PowerPoint shapes, text boxes, icons, lines, arrows, tables, and connectors.
2. Inspect the source image before coding. Note slide aspect ratio, major regions, colors, fonts, labels, and repeated components.
3. For editable reconstruction, copy `scripts/image2ppt.py` from this skill into the task directory and adapt it:
   - Keep helpers such as `I()`, `set_run_font()`, `add_text()`, `style_shape()`, and shape helpers when useful.
   - Replace `build_deck()` content with the layout for the current image.
   - Keep Chinese text in `微软雅黑` unless the source image clearly uses another font.
   - Use grouped helper functions for repeated nodes/cards/lanes; avoid one-off unreadable coordinate blocks when structure repeats.
4. Save the deck with a descriptive `.pptx` filename in the user's working directory.
5. Render or open-check the PPTX and iterate until the slide visually matches the source image closely enough for practical editing.

## Static Image PPT

Use the static path when the user only needs a PPT container for images. Create one blank slide per image, set slide size to a standard widescreen ratio unless the user asks otherwise, and use `slide.shapes.add_picture()` with margins or full-bleed placement.

Do not spend time reconstructing editable shapes unless the user asks for editability or the image is a diagram that clearly benefits from editable PPT objects.

## Editable Reconstruction

Use the editable path for screenshots, workflow diagrams, architecture diagrams, organization charts, product diagrams, and presentation-style infographics.

Implementation preferences:

- Use `python-pptx`; avoid rasterizing the whole slide unless a background/photo must remain an image.
- Set `prs.slide_width` and `prs.slide_height` to match the source aspect ratio.
- Define colors as named constants from sampled or visually matched hex values.
- Build reusable functions for repeated visual units: process nodes, cards, lanes, badges, headers, arrows, callouts.
- Keep text editable with `add_textbox()` or text-bearing shapes.
- Use actual PowerPoint connectors/arrows where possible so users can adjust them after generation.
- Prefer simple geometry and accurate alignment over decorative complexity.
- If exact icon reconstruction is not worth the time, approximate with built-in shapes or simple line icons.

## Running

Typical editable workflow:

```bash
cp /path/to/skill/image2ppt/scripts/image2ppt.py ./image2ppt.py
python3 image2ppt.py --output result-editable.pptx
```

Use the repo's virtual environment when one exists, for example `.venv/bin/python image2ppt.py --output result-editable.pptx`.

Required package: `python-pptx`. Use `Pillow` if the task needs image dimensions, cropping, or color sampling.

## Verification

After generating the PPTX:

- Re-open the PPTX with `python-pptx` to confirm it is readable.
- Render or preview it when possible, using tools such as LibreOffice headless conversion, `qlmanage` thumbnails on macOS, or another available renderer.
- Compare the rendered slide against the source image for layout, text, colors, and missing elements.
- Fix obvious visual mismatches before reporting completion.

## Guardrails

- Do not overwrite the user's original `image2ppt.py` unless explicitly asked.
- Do not claim the result is editable if the main content is only a flattened screenshot.
- Preserve original images as source artifacts; write generated decks as new files.
- If a source image is too low-resolution to read text, state the uncertainty and ask for a clearer image or the missing text.
