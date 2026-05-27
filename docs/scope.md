# Scope

## Current scope

- Open a local folder of JPEG, PNG, or PDF files via the File System Access API
- Image viewer: zoom, pan, contrast, brightness, fine rotation (±45°),
  90° rotation (destructive, writes to disk), fit-to-window, reset view
- Transcribe visible text in up to three languages per image with per-language
  notes; navigate with keyboard arrows or prev/next buttons; thumbnail strip
- Draw rectangular regions of interest; crop files written to project folder
  immediately on draw; delete regions removes crop files from disk
- Export all crops across all images as a flat ZIP
- Re-import externally edited crop files matched by embedded region ID
- Collection-level Item Metadata: identifier, title, description, content
  language, subject language, country, origination date, region, original
  media, data categories, discourse type, dialect, language-as-given,
  license, people and roles
- Save all state to `ro-crate-metadata.json` conforming to RO-Crate 1.1/1.2
  and the LDAC profile; autosave on navigation and after typing pause;
  manual save via Ctrl+S
- Resume from an existing `ro-crate-metadata.json` on folder reopen
- PDF to JPEG page extraction on folder open (PDF.js, CDN on first use)
- Dark/light theme; help modal with workflow summary and keyboard shortcuts

## Non-goals

- OCR / automatic text extraction
- Server-side processing or cloud storage
- Firefox or Safari file-write support (browser API constraint)
- Non-rectangular region shapes
- Undo / redo
- Multi-user collaboration
- IIIF server integration

## Success criteria

| Criterion | Status |
|---|---|
| Researcher can open a local folder and transcribe all images across languages | ✅ met |
| On folder reopen, full state is restored from `ro-crate-metadata.json` | ✅ met |
| `ro-crate-metadata.json` is parseable by an independent RO-Crate tool | ✅ met |
| Region crops are written to the project folder immediately on draw | ✅ met |
| Edited crops can be re-imported and linked back to their source regions | ✅ met |
| No data leaves the researcher's machine during normal operation | ✅ met |
