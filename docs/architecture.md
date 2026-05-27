# Architecture

## Runtime view

### Overview

Picture Book Transcription Editor is a single-page browser application
delivered as one static HTML file with all CSS and JavaScript inlined
(ADR-0001). There is no server, no build step, and no installation. The
application runs entirely in the researcher's browser; all files are read
from and written to a local folder on the researcher's machine.

### Technology stack

| Concern | Choice |
|---|---|
| Language / runtime | Vanilla JavaScript (ES2020+), no framework |
| Deployment | Single static `.html` file, no bundler |
| File I/O | File System Access API — Chrome/Edge 86+ (ADR-0002) |
| Metadata format | RO-Crate 1.1/1.2 + LDAC profile (ADR-0003) |
| PDF rendering | PDF.js 3.11.174, loaded on demand from cdnjs (ADR-0004) |
| ZIP packaging | JSZip 3.10.1, loaded on demand from cdnjs (ADR-0007) |

### Browser constraint

Chrome and Edge 86+ are required for the File System Access API. Firefox
and Safari can render and edit text but cannot write files back to the
project folder. This is a hard constraint (ADR-0002), not a roadmap item.

### Data flow

```
researcher
  │
  ▼
showDirectoryPicker()          ← file-access module
  │  enumerates JPEG/PNG/PDF
  │  extracts PDF pages via PDF.js (on demand)
  │  reads all folder files → auditHandleMap
  │
  ├─ (no existing crate) ──────────────────────────► editor
  │
  └─ (crate found) ──► Folder Audit screen
       │  shows all folder files vs. crate contents
       │  user can add new images or register data files
       ▼
imageFiles[]  ←  in-memory ordered list of {name, FileHandle}
registeredDataFiles[]  ← arbitrary non-image files added via Folder Audit
  │
  ├─► image-viewer             ← renders current image + canvas overlay
  │     zoom / pan / contrast / brightness / rotation
  │
  ├─► region-selection         ← coordinate conversion + region CRUD
  │     writes crop JPEGs to project folder immediately on draw
  │
  ├─► transcription            ← per-image text in up to 3 language slots
  │
  └─► item-metadata            ← collection-level archival fields (crateInfo)
        │
        ▼
      crate-io                 ← serialises all state to ro-crate-metadata.json
        │  autosave on navigation + typing pause; Ctrl+S for manual save
        │  loads existing file on folder reopen to restore full session state
        ▼
  ro-crate-metadata.json       ← written to project folder via File System API
```

No network calls occur during normal operation. CDN library loads
(PDF.js, JSZip) happen only on first use of PDF or ZIP features.

### State model

All per-image state is held in the `data` object (keyed by filename):

```
data[filename] = {
  langs: [{code, text, notes}, {code, text, notes}, {code, text, notes}],
  rotation: Number,         // 0 | 90 | 180 | 270 — structural, stored to disk
  fineRotation: Number,     // -45..45 — visual-only, stored per image
  regions: [{id, x, y, w, h, editedCropName}]
}
```

Collection-level state lives in `crateInfo` (identifier, title, people,
license, etc.) and `globalLangCodes[3]` (ADR-0006).

Folder Audit state (v02+):
- `registeredDataFiles[]` — `{name, handle}` entries for non-image files the
  user explicitly registered via the Audit screen; serialised as bare
  `@type: File` + `hasPart` entries in the RO-Crate.
- `auditCrateLoadedFiles` — `Set<string>` of filenames already present in the
  loaded crate; used to compute the "not in crate" badge count in the audit UI.
- `auditHandleMap` — `{[name]: FileHandle}` for every file found in the folder
  on open; used to resolve handles when the user clicks Register.

The `dirty` flag and `autoSaveTimer` are managed by `crate-io`; a 3-second
typing pause or any navigation triggers an autosave.

### Extension points

- **Additional image transforms** (e.g. invert, grayscale) can be added to
  `image-viewer` as new slider rows and CSS filter terms without touching
  other modules.
- **Additional archival fields** can be added to `crateInfo` and the Item
  Metadata modal without touching transcription or region modules.
- **Additional CDN libraries** follow the on-demand pattern established in
  ADR-0004: lazy `<script>` injection with an error toast on failure.

---

## Software architecture

Each Bower module corresponds to a distinct functional block in the single
HTML file, separated by `// ═══` comment section headers in the source.

### file-access

**Purpose:** Entry point. Handles the OS-level folder picker, enumerates
image files (JPEG/PNG), detects and extracts PDF sources to JPEG pages,
builds the ordered `imageFiles` array, and (v02+) enumerates all folder
files to populate the Folder Audit screen.

**Data concern:** owns `dirHandle` (the live File System Access directory
reference), `imageFiles[]`, `pdfSources[]`, and (v02+) `registeredDataFiles[]`,
`auditCrateLoadedFiles`, `auditHandleMap`. All other modules that need to read
or write files receive `dirHandle` indirectly — it is the single shared gateway
to the project folder.

**Features:** folder-open, image-enumeration, pdf-extraction, folder-audit (v02+)

**Dependencies:** none (entry point)
**Consumed by:** image-viewer, region-selection, crate-io, crop-transfer

---

### image-viewer

**Purpose:** Renders the current image in the viewport with pan, zoom,
contrast, brightness, fine rotation, and 90-degree rotation. Manages the
canvas overlay that draws region bounding boxes and the in-progress
selection rectangle. Zoom/contrast/brightness settings persist across
image navigation.

**Data concern:** owns view state — `zoom`, `contrast`, `brightness`,
`rotation`, `fineRotation`, `panX`, `panY`. The 90-degree rotation is
destructive: it writes a new image file to disk and is therefore both a
view operation and a file-access operation.

**Features:** image-transform-controls, pan-and-zoom, rotation-and-flip,
canvas-overlay

**Dependencies:** file-access (image file handles for loading and for the
destructive rotation write), region-selection (region geometry for overlay
drawing)
**Consumed by:** region-selection (transform state for coordinate conversion)

---

### region-selection

**Purpose:** Manages the selection mode UI (crosshair cursor, drag-to-draw
rectangle), converts viewport coordinates to original-image-pixel
coordinates (ADR-0005), stores region geometry per image, writes crop JPEG
files to the project folder immediately on region creation, and handles
region deletion (removes the crop file from disk).

**Data concern:** owns `data[filename].regions` — the array of
`{id, x, y, w, h, editedCropName}` objects. The `id` is an 8-character
random alphanumeric embedded in the crop filename, enabling round-trip
matching.

**Features:** select-mode-toggle, coordinate-conversion, region-crud,
crop-write

**Dependencies:** file-access (dirHandle for writing/deleting crop files),
image-viewer (transform state: zoom, pan, rotation for vpToImg/imgToVp)
**Consumed by:** crate-io (region data serialised to RO-Crate),
crop-transfer (crop filenames and IDs)

---

### transcription

**Purpose:** Manages the three-language transcription UI per image —
language tab switching, stacked view, language code input (with global
propagation across all images per ADR-0006), text area, and notes field.

**Data concern:** owns `data[filename].langs[3]` (each entry: `{code,
text, notes}`) and the `globalLangCodes[3]` array.

**Features:** language-tabs, stacked-view, lang-code-global-propagation,
text-and-notes-input

**Dependencies:** file-access (imageFiles list for global code propagation
across all images)
**Consumed by:** crate-io (transcription data serialised to RO-Crate)

---

### item-metadata

**Purpose:** Captures collection-level archival metadata via the Item
Metadata modal. Fields include identifier, title, description, content
language, subject language, country, origination date, region, original
media, data categories, discourse type, dialect, language-as-given,
license, and people/roles (17-role vocabulary).

**Data concern:** owns the `crateInfo` object.

**Features:** item-metadata-modal, license-selector, people-and-roles-editor

**Dependencies:** none (UI only; no internal data dependencies)
**Consumed by:** crate-io (crateInfo serialised into the RO-Crate root
dataset node)

---

### crate-io

**Purpose:** Serialises the full in-memory application state to RO-Crate
1.1/1.2 JSON (ADR-0003) and writes it to `ro-crate-metadata.json` in the
project folder. Loads and hydrates state from an existing file on folder
open. Owns the dirty flag and autosave timer.

**Data concern:** owns the `ro-crate-metadata.json` file and the
dirty/autosave state machine. Reads from all other modules' data; produces
the canonical persisted state.

**Features:** ro-crate-build, ro-crate-save, ro-crate-load,
dirty-state-and-autosave

**Dependencies:** file-access (dirHandle), region-selection (region data),
transcription (transcription data), item-metadata (crateInfo)
**Consumed by:** none (terminal; writes to disk)

---

### crop-transfer

**Purpose:** Exports all region crops across all images as a flat ZIP of
JPEGs (using JSZip, ADR-0007). Re-imports a folder of externally edited
crop files, matching them by the embedded region ID in the filename, copying
them into the project folder, and recording the `editedCropName` in the
region metadata entry.

**Data concern:** no new state; reads region geometry and crop filenames
from `data`, writes `editedCropName` back to `data[filename].regions[i]`.

**Features:** crop-zip-download, edited-crop-import

**Dependencies:** file-access (dirHandle, image file handles),
region-selection (region data and crop filenames)
**Consumed by:** crate-io (editedCropName values serialised into RO-Crate)
