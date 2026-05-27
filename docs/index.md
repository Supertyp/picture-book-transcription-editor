# Picture Book Transcription Editor — Project Index

## Scope
See [scope.md](scope.md) — all success criteria ✅ met.

## Architecture
See [architecture.md](architecture.md) — runtime view and 7-module software architecture.

## UI
See [ui.md](ui.md) — navigation, screens, layout grammar, and interaction patterns.

## Design decisions
See [adr/index.md](adr/index.md) — 7 ADRs covering architecture and key technical choices.

## Modules
All modules implemented directly in the HTML source (not Bower-scaffolded). Each corresponds to a `// ═══` section in the file.

| Module | Purpose | Status |
|---|---|---|
| file-access | Folder picker, image enumeration, PDF extraction | ✅ complete |
| image-viewer | Image transforms, canvas overlay | ✅ complete |
| region-selection | Region drawing, coordinate conversion, crop writing | ✅ complete |
| transcription | Language tabs, text/notes, global language codes | ✅ complete |
| item-metadata | Collection-level archival metadata modal | ✅ complete |
| crate-io | RO-Crate serialisation, save/load, autosave | ✅ complete |
| crop-transfer | ZIP export and edited-crop import | ✅ complete |
