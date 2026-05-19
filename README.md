# Picture Book Transcription Editor

A single-file browser-based tool for transcribing text from historical images and marking regions of interest, with all metadata saved as a native [RO-Crate](https://www.researchobject.org/ro-crate/).

## What it does

Open a folder of JPEG, PNG or PDF files, transcribe the text you see in up to three languages per image, draw bounding boxes around areas of interest (stamps, seals, drawings), and export or import edited crops. Everything is tracked in a standard `ro-crate-metadata.json` file written directly into your project folder.

## Getting started

1. Open `picture-book-transcription-editor.html` in **Google Chrome** or **Microsoft Edge** (version 86 or later).
2. Click **Open Folder** and select the folder containing your images or PDFs.
   - PDFs are split into one JPEG per page automatically (requires an internet connection the first time, to load the PDF.js library).
   - If a `ro-crate-metadata.json` already exists in the folder, your previous work is restored instantly.
3. Transcribe, annotate, and save.

> Firefox and Safari can open the tool and edit text, but cannot write files directly — you would need to copy the generated JSON manually.

## Features

### Image viewer
- **Zoom** (10–400 %), **Contrast**, **Brightness**, and **Fine Rotate** (±45°) sliders so you can make faded or skewed text legible without altering the source file.
- **Rotate 90°** for images captured sideways.
- **Pan** by clicking and dragging anywhere on the image; scroll wheel zooms.
- **Fit** button to reset the view.

### Transcription
- Three language tabs per image, each with an ISO 639-3 language code field (e.g. `eng`, `tok`, `fra`). Language codes are global — set once and applied to all images.
- A free-text **Notes** field per language for uncertain readings or editorial comments.
- **Stacked view** toggle to see all three languages at once.
- Thumbnail strip at the bottom for quick navigation; completed images are highlighted in green.

### Region selection and crops
- Click **Select Region**, then drag a rectangle on the image to mark an area. The crop is saved immediately as a JPEG in your project folder.
- Click a region tag to highlight its bounding box; click **✕** on a tag to delete the region and remove the file.
- **Download Crops** exports all regions across all images as a flat ZIP of JPEGs for editing in any image editor.
- **Import Crops** re-imports a folder of edited files, matching them by their embedded ID. Edited regions are shown with a green **✓ edited** indicator.

### Metadata
- **Item Metadata** dialog captures archival fields: identifier, title, description, content language, subject language, country, origination date, region, original media, data categories, discourse type, dialect, people & roles, and license.
- Supported licenses: CC BY 4.0, CC BY-SA 4.0, CC BY-NC 4.0, CC0 1.0, or custom.

### Saving
- Metadata is saved automatically on navigation and after a short typing pause.
- Press **Ctrl+S** or click **Save** for an immediate write.
- A status indicator (dot + label) shows whether the file is up to date or has unsaved changes.

### RO-Crate output
All data is written to `ro-crate-metadata.json` in the open [RO-Crate 1.1](https://www.researchobject.org/ro-crate/specification/1.1/) format. The file records every image, transcription, region, crop and edited file with full provenance. It can be read by any RO-Crate-compatible tool and shared or archived alongside the images.

## Keyboard shortcuts

| Key | Action |
|---|---|
| `← →` | Previous / next image |
| `1` `2` `3` | Switch language tab |
| `Ctrl+S` | Save RO-Crate |
| `Esc` | Exit region draw mode |
| Scroll wheel | Zoom in / out |
| Click + drag | Pan image (pan mode) |

## Browser requirement

Requires **Chrome or Edge 86+** for the [File System Access API](https://developer.mozilla.org/en-US/docs/Web/API/File_System_API) used to read and write files directly. No server, no install, no data leaves your machine.

## No dependencies to install

Everything runs client-side. The only external resources loaded are:
- [IBM Plex fonts](https://fonts.google.com/specimen/IBM+Plex+Mono) (Google Fonts, for the UI)
- [PDF.js](https://mozilla.github.io/pdf.js/) (loaded on demand when a PDF is opened)
