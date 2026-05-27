---
id: ADR-0002
title: File System Access API for local file I/O
status: accepted
date: 2026-05-23
---

## Context

The tool needs to read images from a local folder and write files back to
that same folder (crop JPEGs, `ro-crate-metadata.json`). This must happen
without a server and without requiring the researcher to manually move files.
The write-back requirement distinguishes this from a simple file-input workflow.

## Decision

The application uses the browser File System Access API
(`window.showDirectoryPicker`, `FileSystemDirectoryHandle.getFileHandle`,
`FileSystemWritableFileStream.write`) exclusively for all file I/O. This
API allows the browser to hold a live handle to a directory and read/write
files within it for the duration of the session, with a one-time user
permission grant.

## Consequences

- **Positive:** files are written directly into the project folder; crops
  appear in place, `ro-crate-metadata.json` is saved in place.
- **Positive:** the session can resume from the same folder on reopen;
  `tryLoadRoCrate` reads the existing metadata file on folder open.
- **Negative:** the API is only available in Chrome and Edge 86+. Firefox
  and Safari cannot write files. This is surfaced via a warning toast and
  in the help modal.
- **Negative:** the permission grant is session-scoped; the researcher
  must re-grant on every browser restart.

## Alternatives considered

- **`<input type="file" webkitdirectory>`:** can read a folder but cannot
  write back to it. Ruled out.
- **IndexedDB + export on demand:** loses the files-live-in-project-folder
  property that makes the RO-Crate self-contained. Ruled out.
- **Electron / Tauri:** native file access without browser constraints,
  but requires installation. Ruled out per ADR-0001.
