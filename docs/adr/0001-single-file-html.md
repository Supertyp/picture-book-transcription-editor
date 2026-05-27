---
id: ADR-0001
title: Single-file HTML application with no build step
status: accepted
date: 2026-05-23
---

## Context

The tool targets archival researchers who are not software engineers. The
primary distribution requirement is zero-friction deployment: no npm, no
server, no Python environment. The tool must run from a local file opened
in a browser. Privacy requirements mean no cloud hosting is acceptable as
a default. The application is also small enough (~1400 lines) that a
single-file approach does not create maintainability problems at current scale.

## Decision

The entire application — HTML structure, CSS, and JavaScript — is inlined
in a single `picture-book-transcription-editor.html` file. No build step,
no bundler, no package manager, no external dependencies at load time
(CDN libraries are loaded on demand when specific features are first used;
see ADR-0004 and ADR-0007). The file can be opened by double-click or
dragged into a browser.

## Consequences

- **Positive:** zero installation friction; works fully offline for all
  core features; trivially sharable (email, USB drive, GitHub release).
- **Positive:** the full source is inspectable by any researcher who opens
  the file in a text editor.
- **Negative:** all CSS and JS must coexist in one file; as the application
  grows, this imposes a discipline cost.
- **Negative:** no module system or tree-shaking; all code is always loaded.
- **Constraint:** any feature that requires a package must use the on-demand
  CDN pattern (ADR-0004) rather than bundling.

## Alternatives considered

- **Node.js / Electron desktop app:** would allow a proper build system and
  native file access, but requires installation. Ruled out on friction grounds.
- **Vite SPA with build step:** clean but requires Node.js and adds build
  complexity for a single-contributor tool. Ruled out.
- **Hosted web app:** contradicts the privacy and data-sovereignty constraint.
  Ruled out.
