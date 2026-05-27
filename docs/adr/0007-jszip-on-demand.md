---
id: ADR-0007
title: JSZip loaded on demand from CDN for crop ZIP export
status: accepted
date: 2026-05-23
---

## Context

The crop download feature packages all region crops into a ZIP file for
bulk delivery to the researcher. The browser has no native ZIP API.
JSZip (~100 KB minified) is the standard choice. As with PDF.js (ADR-0004),
inlining it increases the base file size for all users.

## Decision

JSZip 3.10.1 is loaded from cdnjs at the moment the researcher first clicks
"Download Crops" (`downloadCrops()`). If the load fails, the function throws
and the user receives no ZIP. The library is cached by the browser after
first load.

## Consequences

- **Positive:** no impact on initial file size or load time.
- **Positive:** follows the same on-demand CDN pattern as ADR-0004,
  keeping the architecture consistent.
- **Negative:** an internet connection is required on first use of the
  ZIP download feature.
- **Note:** the crop import path (`importCrops`) requires no library;
  it uses only the File System Access API.

## Alternatives considered

- **Bundle JSZip inline:** adds ~100 KB to the base file size. Ruled out
  per ADR-0001 principle of keeping the default file small.
- **Native multi-file download without ZIP:** browsers throttle or block
  sequential anchor download triggers for bulk files. Ruled out.
