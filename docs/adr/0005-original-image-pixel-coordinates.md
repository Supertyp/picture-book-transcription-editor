---
id: ADR-0005
title: Region coordinates stored in original-image-pixel space
status: accepted
date: 2026-05-23
---

## Context

When a researcher draws a region on the image viewer, the coordinates of
the rectangle must be stored independently of the current view state
(zoom, pan, fine-rotation) and be meaningful in the RO-Crate output
(where they appear as `spatialCoverage/geo/box` values). The viewer applies
arbitrary CSS transforms; stored coordinates must survive those transforms
and round-trip correctly when the file is reloaded.

## Decision

Region bounding boxes are stored as integer pixel values (`x`, `y`, `w`,
`h`) in the coordinate space of the original (unrotated, un-zoomed) image.
Two conversion functions handle the mapping at interaction time: `vpToImg`
transforms a viewport-relative pointer position into original-image pixels
(undoing pan, zoom, and rotation), and `imgToVp` inverts this for overlay
drawing. The 90-degree rotation operation is destructive (writes a new image
file to disk and resets `rotation` to 0), so stored coordinates always
reference the on-disk image.

## Consequences

- **Positive:** stored coordinates are stable across zoom/pan/brightness
  changes; geometry in `ro-crate-metadata.json` is always interpretable
  given only the image file.
- **Positive:** consistent with how RO-Crate and IIIF express image regions
  (pixel bounding boxes relative to the source image).
- **Negative:** the `vpToImg` / `imgToVp` math must correctly account for
  all active transforms; bugs produce silently-wrong crop geometry.
- **Constraint:** fine rotation (+-45 degrees) is visual-only and does not
  affect stored coordinates. Crops are extracted axis-aligned from the
  original image.

## Alternatives considered

- **Normalised 0-1 coordinates:** resolution-independent but loses
  sub-pixel precision for small regions. Ruled out.
- **Viewport-relative coordinates:** meaningless after any zoom/pan change.
  Ruled out.
