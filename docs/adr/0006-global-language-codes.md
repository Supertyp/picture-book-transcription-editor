---
id: ADR-0006
title: Language codes are global across all images per slot
status: accepted
date: 2026-05-23
---

## Context

The application provides three language slots per image. A researcher
transcribing a collection of 200 images in English and Tok Pisin would
otherwise need to set the language code on every single image. The question
is whether the language code is a per-image property or a collection-level
property.

## Decision

Each of the three language slots has a single global language code that
applies to all images in the collection. Setting slot 1 to "eng" writes
"eng" to every image entry in memory that had the previous slot-1 code.
The `onLangChange` function propagates the change immediately across all
`data[filename].langs[i].code` values. The code is still stored per image
in `ro-crate-metadata.json` (RO-Crate requires it), but the UI treats it
as a global setting.

## Consequences

- **Positive:** drastically reduces repetitive input for homogeneous
  collections (the common case).
- **Positive:** the global code is shown as a badge on each tab, giving
  the researcher a constant reminder of the active language.
- **Negative:** changing a global code mid-session retroactively updates
  all images that had the old code in that slot, which could be surprising
  for mixed-language collections.
- **Constraint:** per-image language overrides are stored in the data
  model but there is no UI for setting a per-image code that differs from
  the global value.

## Alternatives considered

- **Fully per-image language codes:** correct for heterogeneous collections
  but imposes repetitive entry for the common homogeneous case. Ruled out.
- **Collection-level default with per-image override UI:** the right
  long-term model but adds UI complexity. Deferred.
