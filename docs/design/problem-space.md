# Problem Space: Picture Book Transcription Editor

## The problem and who has it

Archival researchers, linguists, and heritage collection managers working
with historical image collections — manuscripts, picture books, missionary
photographs, field notebooks — need to capture what text appears in images,
in what languages, and mark regions of interest for closer study or separate
archiving.

These researchers work under strict privacy and data-sovereignty requirements:
source materials may be restricted, participants may be identifiable, and
data must not leave the researcher's machine. Their collections are local
folders of images, not hosted repositories.

The transcription task is multilingual by nature. A single page may carry
text in English, Tok Pisin, and a vernacular language simultaneously.
Regions of interest — stamps, seals, drawings, marginal notes — need to be
extractable as separate, citable image artefacts.

The output must be usable beyond the immediate project: depositable in an
archive, shareable with collaborators, or ingestible by downstream tools.

## Why current alternatives are insufficient

- **General-purpose transcription tools** (Transkribus, FromThePage, IIIF
  annotation tools) require server infrastructure, institutional accounts,
  or cloud connectivity. This conflicts with privacy requirements and creates
  vendor lock-in via proprietary metadata formats.
- **Spreadsheet-based workflows** capture text but have no spatial metadata
  for regions of interest, no image viewer, and produce metadata that cannot
  be round-tripped into archival systems.
- **Desktop image viewers with note-taking** (Preview, GIMP, IrfanView)
  provide no structured metadata output, no multi-language support, and no
  concept of linking a note to a specific spatial region.
- **IIIF-based annotation systems** are architecturally correct but require
  a IIIF server, conflicting with the no-server constraint, and have high
  setup friction for field researchers.

None of these approaches produces a self-contained, open-standard metadata
file that lives alongside the images and can be deposited as-is.

## Success criteria

1. A researcher can open a folder of images or PDFs on a local machine using
   a standard browser, with no installation or server required.
2. Text can be transcribed in up to three languages per image, with notes for
   uncertain readings.
3. Rectangular regions of interest can be marked; crop files appear in the
   project folder immediately.
4. The researcher can close the browser and reopen it later, resuming exactly
   where they left off.
5. The resulting `ro-crate-metadata.json` is parseable by any independent
   RO-Crate 1.1/1.2 compatible tool and can be deposited alongside the images.
6. No data leaves the researcher's machine during normal operation.

## Scope boundaries

In scope: local file access via browser, multilingual text transcription,
spatial region annotation, crop export and re-import, RO-Crate 1.1/1.2
output, collection-level archival metadata (LDAC profile), image viewer
with common transforms.

Out of scope: OCR, server-side processing, cloud storage, Firefox/Safari
file-write support, non-rectangular regions, undo/redo, multi-user
collaboration.

## Constraints

- **Browser:** Chrome/Edge 86+ required for the File System Access API
  (ADR-0002). Hard constraint; not a preference.
- **Single-file deployment:** the tool ships as one `.html` file (ADR-0001).
  All additions must remain compatible with this model.
- **No build system:** no package manager, bundler, or test runner. Changes
  are made directly to the HTML file.
- **Interoperability anchor:** RO-Crate format (ADR-0003) is non-negotiable.
  Schema deviations that break conformance with the spec are out of scope.
