---
id: ADR-0004
title: PDF.js loaded on demand from CDN
status: accepted
date: 2026-05-23
---

## Context

PDF support is needed because researchers often receive scanned documents
as PDF files. PDF.js is the standard browser-side library for rendering
PDF pages to canvas. At ~1 MB, bundling it inline would roughly double the
application file size for every user, even those who never open a PDF.

## Decision

PDF.js 3.11.174 is loaded from cdnjs at the moment a PDF file is first
detected in the opened folder (`loadPdfJs()`). The worker script is also
loaded from cdnjs at the same version. If the CDN load fails, the user
receives a warning toast and PDF processing is skipped. After first load,
the browser caches the library for subsequent sessions.

## Consequences

- **Positive:** the default HTML file size stays small; image-only users
  pay no cost.
- **Positive:** version is pinned (3.11.174), so behaviour is reproducible.
- **Negative:** an internet connection is required the first time a PDF is
  processed in a new browser profile. The help modal documents this.
- **Negative:** if cdnjs is unavailable or the pinned version is removed,
  PDF extraction degrades to a user-visible warning.

## Alternatives considered

- **Bundle PDF.js inline:** keeps the tool fully offline-capable but adds
  ~1 MB to the file for all users. Ruled out on file-size grounds.
- **Require pre-converted JPEGs:** shifts the conversion burden to the
  researcher. Ruled out.
- **Self-host PDF.js:** requires a server, contradicting ADR-0001. Ruled out.
