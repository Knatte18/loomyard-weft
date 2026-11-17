# Render HEDDLE.md by the place each entry lands in

`HEDDLE.md` was hard to read.
It grouped entries by target, gave each a From, Order and Targets list with its full text in a fenced block, and spent a heading and a "No entry." line on every empty target.
A reader scrolled through every shipped entry's whole text, and could not tell what lym sets from what the repository sets without reading each From line.

`RenderOverview` in `internal/heddleengine/overview.go` now renders it as follows:

- The intro defines the four sources: **repo**, **lym**, **lym, overridden** and **derived**.
  Bold marks the ones this hub sets, which are repo and overridden.
- An "At a glance" table comes first, with one row per entry and place.
  Its columns are where the entry lands, its source, its slug and a one-line summary.
  A text entry is summed up by its first heading, or its first line when it has none; a JSON-object entry by the keys it sets.
- A section follows for each file or prompt that some entry reaches.
  A place no entry reaches is left out.
  A shared target such as `agents-file` has no section of its own; its entries show under each file it renders into.
- A text or prompt section groups its entries by source: this repository, lym overridden by this hub, derived from the hub's config, then lym.
  The first three show full text.
  Only lym's own unchanged entries are folded under a `<details>` summary.
- A JSON-object section such as `.claude/settings.local.json` shows the object its entries fold to, as a `jsonc` block.
  A comment above each run of keys names the entries that set it and their sources.
- The layers' override and derived warnings show as notes under the intro instead of inside one section.
  An entry whose text holds no JSON object is named in a note and left out of the folded object.

`internal/heddleengine/doc.go` describes the new layout.
`overview_test.go` is rewritten to pin the table, the source groups, the shared-target fan-out, the widened fence, the escaped table pipe, the folded settings object and the omitted empty place.
The CLI integration and replayer tests now match table rows instead of `###` headings.

## Design decisions

- What this hub sets comes before what lym ships, and lym's own text is the only part folded, so the reader sees their own changes first.
- A JSON-object place shows the folded result, which is what the worktree file receives, and marks the setters in comments.
  It does not repeat each entry's object.
- The settings object is produced by the same `foldObjects` the delivery uses, so the overview cannot drift from what is written.

## To check by hand

- Open a rendered `HEDDLE.md` from this hub's real heddle store, in a markdown viewer that supports `<details>`.
  Confirm that the layout reads well and that the folded `lym` entries expand.
- No regression check ran for this change.
