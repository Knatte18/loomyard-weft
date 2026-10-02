# Card 6 — render-roadmap

**Edit:**
- `internal/boardengine/render.go`
- `internal/boardengine/layer.go`
- `internal/boardengine/store.go`
- `internal/boardengine/render_test.go`
- `internal/boardengine/layer_test.go`
- `internal/boardengine/store_test.go`

**Intent:**
Render the README like `manifest/roadmap.md` — one section per tier, one `1.` item per entry — and give each design doc a header saying what it belongs to (Decisions 3, 18–21).

`layer.go`: `RenderOrder` returns README order: non-`done` entries by tier (1, 2, 3), then `done` entries;
within each section, dependency layer then id.
`ComputeLayers` keeps computing the layer used for that ordering; layer letters are never rendered.

`render.go`:
- The section names and their one-line meanings live here alone, as the only place tier numbers map to words: `Planned` (tier 1, concretized and claimable), `Next Up` (tier 2, planned next but not concretized), `Someday` (tier 3, loose ideas), and `Done` (finished, awaiting `lyx board prune`).
- The README is `# Board`, a one-line intro, then `## Planned`, `## Next Up`, `## Someday`, `## Done`, each heading followed by its meaning line.
  A tier section renders its heading and meaning even when empty; `## Done` is omitted when no entry is done.
  The intro and each meaning line are a single sentence on a line of their own, and the renderer never hard-wraps, so the generated markdown follows semantic line breaks.
  A `done` entry renders under `## Done` whatever its tier; every other entry, `abandoned` included, under its tier.
- Each entry is one numbered item, exactly:

  ```
  1. **<title>** — `<slug>` · <type>[ · <status>]
     <brief>[ [design](<design-prefix><slug>.md)]
     After `<dep>`, `<dep>`.
  ```

  The design link appears only when the entry has a body; the second line is omitted when there is neither brief nor body;
  the `After` line appears only when the entry has dependencies.
  No per-entry headings, no ids, no layer letters.
- A design doc is `# <title>`, a metadata line `` `<slug>` · <tier section name> · <type>[ · <status>] ``, a `Depends on:` line when the entry has dependencies — linking each dependency's design doc, or naming the dependency in backticks when it has no body — then the body byte-for-byte as stored, with no added line breaks.
- The package's own file comment describes the new shape.

`store.go`: `ListTasksBrief` returns entries in `RenderOrder`'s order, falling back to store order when layer computation fails, so `list`, `find` and card 7's `--text` share the README's order.

Tests:
- `render_test.go`: a README golden over a fixture holding an entry per tier, an empty tier 2, a `done` entry, an `abandoned` tier-3 entry with its status tag, one entry with a body and one with dependencies, and a two-layer dependency chain inside one section ordered by layer;
  a second golden with no `done` entry asserting `## Done` is absent;
  design-doc goldens for the header and metadata line, for `Depends on:` with one linked and one named dependency, and for a multi-line body appearing byte-identical.
  The old bucket-heading and status-variant cases are replaced by these.
- `layer_test.go`: `RenderOrder` sorts tier before layer and puts `done` last.
- `store_test.go`: `ListTasksBrief` and `Find` return README order.

**ImpactSummary:** README and design-doc output change wholesale on the next render of every board; `list` JSON order changes to README order, which reaches only `internal/boardcli`'s `list` (the sole production caller of `ListTasksBrief`), since no stencil reads `lyx board list`; no API signature changes.
