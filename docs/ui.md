# UI Documentation — Picture Book Transcription Editor

## Navigation

The application has three top-level views, each occupying the full viewport:

| View | ID | When shown |
|---|---|---|
| Welcome / start screen | `#welcome` | On load; no folder open |
| Folder Audit | `#folderAudit` | After folder open **when an existing `ro-crate-metadata.json` is present** |
| Editor | `#editor` | After audit is dismissed; or directly after folder open when no crate exists |

Transitions:
- **Open Folder → (no crate)** — skip audit, go directly to editor.
- **Open Folder → (crate found)** — show audit gate; user continues to editor.
- **Audit → Continue / Add** — hides audit, shows editor.
- **Editor** — no back-navigation to audit; re-opening the folder restarts the flow.

## Screens

### Welcome

Full-viewport centred card. Contains the tool title, a workflow summary (5 numbered steps), browser requirement note, and a primary CTA **📂 Open Folder to Begin**.

### Folder Audit

Appears after reopening a folder that already has `ro-crate-metadata.json`. Purpose: surface files that have been added to the folder manually and are missing from the existing RO-Crate.

**Header strip:**
- Folder name (amber monospace)
- File count summary: `N files · M in RO-Crate · K not in crate`
- **+ Add all missing (K)** — primary button; disabled when K = 0; adds all untracked image/PDF files to `imageFiles[]` and opens the editor at the first new file
- **Continue to Editor →** — success button; proceeds to editor without adding anything

**File list (scrollable):**
Each row is a four-column grid: icon | filename | status badge | action.

| File type | Icon | Status badges | Actions |
|---|---|---|---|
| `ro-crate-metadata.json`, `ro-crate-preview.jpg` | 🔒 | `system` (dim) | none |
| Crop files (`*_crop_*.jpg`) | ✂️ | `crop` (dim) | none |
| Image files (jpg/png) — in crate | 🖼️ | `✓ in RO-Crate` (green) | none |
| Image files — not in crate | 🖼️ | `✕ new file` (amber) | **+ Open** |
| PDF files — in crate | 📋 | `✓ in RO-Crate` (green) | none |
| PDF files — not in crate | 📋 | `✕ new file` (amber) | **+ Open** |
| Other files — in crate | 📄 | `✓ in RO-Crate` (green) | none |
| Other files — not in crate | 📄 | `not in crate` (dim) | **Register** |

**+ Open** (image/PDF): adds the file to the editor's `imageFiles[]`, saves the crate, opens the editor navigated to that file.

**Register** (data files): adds the file to `registeredDataFiles[]` which causes it to be written as a bare `@type: File` entity and `hasPart` reference in the next crate save. The button updates to `✓ registered` inline without leaving the audit screen.

System-row items (🔒 and ✂️) are dimmed to 45% opacity and have no hover state.

### Editor

Split-pane layout: image pane (55 %) + text pane (45 %). Described in `docs/architecture.md` — runtime view.

## Layout grammar

- All three top-level views share the same `<header>` above them.
- `#welcome` and `#folderAudit` are `display: flex; flex-direction: column` and occupy `flex: 1` of the body height.
- `#editor` is `display: flex; flex-direction: column` with the panes row below the toolbar.
- Modal overlays (Crate Info, Help, PDF progress, JSON export) use fixed positioning with a semi-transparent backdrop.

## Interaction patterns

- **Inline badge update** — clicking Register updates the row's badge in-place without a page transition. The + Add all missing count in the header button refreshes accordingly.
- **Direct editor entry** — clicking + Open immediately switches to the editor and navigates to that file; the audit list is not revisited (re-open the folder to audit again).
- **No mandatory review** — the audit gate always offers a "Continue to Editor →" escape hatch. It never blocks access to previously-loaded images.

## Visual language

Design tokens are defined as CSS custom properties in `:root` (dark) and `:root.light` (light theme). Key tokens:

| Token | Role |
|---|---|
| `--accent` | Primary interactive / active state (blue) |
| `--accent2` | Warning / new / amber highlights |
| `--accent3` | Success / in-crate / green |
| `--text-dim` | Secondary labels, system-row dimming |
| `--border` | Borders, inactive thumbnails |
| `--mono` | All code-like text (filenames, badges, counters) |
| `--sans` | Body text, buttons |

Audit badges use the same semantic colour mapping: green (`--accent3`) for confirmed/in-crate, amber (`--accent2`) for new/untracked, muted (`--text-dim`) for system/internal.
