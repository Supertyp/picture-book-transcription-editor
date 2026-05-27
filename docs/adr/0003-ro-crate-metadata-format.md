---
id: ADR-0003
title: RO-Crate 1.1/1.2 as the metadata serialisation format
status: accepted
date: 2026-05-23
---

## Context

The tool needs to persist all session state — image list, per-image
transcriptions, region geometry, crop provenance, and collection-level
metadata — in a format that survives browser restarts and can be deposited
in an archive or shared with collaborators without the tool installed.
The format must handle both structural metadata (file relationships) and
the domain-specific archival fields used in language documentation (content
language, subject language, discourse type, dialect, etc.).

## Decision

All metadata is serialised as a single `ro-crate-metadata.json` file
conforming to the RO-Crate 1.1/1.2 specification and the LDAC (Language
Data Commons Australia) profile. The file uses a JSON-LD `@graph` array
where each node represents an entity: the dataset root, each image file,
each crop file, each edited crop, and each person contributor. Transcripts
are represented as inline `Transcript` objects on each image node.

## Consequences

- **Positive:** the output file can be opened by any RO-Crate compatible
  tool without the transcription editor.
- **Positive:** the LDAC profile covers the full vocabulary of language
  archival metadata fields that the target users need.
- **Positive:** the graph structure naturally represents provenance chains
  (crop to edited crop to source image) without a bespoke schema.
- **Negative:** JSON-LD graph syntax is more verbose than a flat schema;
  the serialisation logic (`buildRoCrateJson`) is correspondingly complex.
- **Negative:** loading requires traversing the graph to reconstruct
  per-image state; `importFromRoCrate` must handle multiple node types.

## Alternatives considered

- **Bespoke flat JSON:** simpler to read/write but produces no
  interoperable output. Ruled out.
- **IIIF Manifest:** server-URL-centric and lacks the LDAC archival field
  vocabulary. Ruled out.
- **CSV sidecar files:** cannot represent region geometry or provenance.
  Ruled out.
