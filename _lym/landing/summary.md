# Board README: beam marker, no empty sections, flat headings

The rendered board files (`README.md`, each project file) carried a lot of scaffolding:
a meaning line under every heading, a `Tasks` and a `Notes` level above the real sections,
`_None._` placeholders, and collapsible `<details>` blocks that a re-render reset anyway.
This change strips them so the entries stand out, and marks beam projects, which the old render only flagged with a bare `beam` line.
The layout was tried out with the operator in a sampler session and these are the choices kept.

## What changes in the rendered files

- A beam project's heading ends in ` · BEAM`, after the title and the member count, in `README.md` and in every project file.
  Its Done row ends the same way.
  The separate `beam` line under the brief is gone.
- A section with no entry renders nothing: no heading, no table, no `_None._`.
  A project's empty Ready no longer shows, and `Own entries` appears only when the project has own entries beside its sub-projects.
- The `Tasks` and `Notes` headings are gone.
  Running, Ready, each dependency layer, Independent, Draft, each note type section and Done are sections of their own, at one level.
- No section carries a one-line meaning under its heading.
- `README.md` renders each project as a plain level-two heading linking its file, not a `<details open>` block.

## Design decisions

- `internal/boardengine/render.go`: `readmeSection` (name plus meaning) is replaced by plain heading constants and `layerHeading`.
  `entrySections` builds every section through one local helper that skips an empty entry list,
  so the "no empty sections" rule has one home rather than a check per section.
- `beamSuffix` is the single place the marker is added; the heading, the file header and the Done row all use it.
- `collapsibleProjectSection` and its `html` import are removed; the README calls `projectSection` directly.
- Section names, their order and `RenderOrder` are unchanged, so the `layer` and section fields of `lym board list` and `find` read as before.
  `view.go` only switches to the renamed constants.
- `render_test.go` and `view_test.go` are rewritten to the new layout, and the stale layout claims in `doc.go` and `docs/overview.md` are updated.

## To check by hand

- Run `lym board render` on a real board and read `README.md` and a project file:
  a beam project, a project with only notes, a project with sub-projects and own entries, and a project with Done entries.
  Anything that relied on the `Tasks` or `Notes` heading, or on a `<details>` block, in a link anchor or an external reader of these files, would break.
- Tests were not run as part of writing this description.

## Merge danger

**Door:** two-way
**Blast radius:** board
