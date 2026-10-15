# Fixer report — plan, round 1

## Changes

- **F1** (`04-forkaudit-card-arm.md`), fixed as proposed: the `card packs` column renders `-` for a fork with no card-pack call and no traced card pack line (no `card-traces.tsv` row, or a row with count 0), `<calls>/<traced>` for a fork with a row and a call or a nonzero count, and `<calls>/-` for calls with no row.
  The agreeing-layouts `fork-file` row's `-` and the fork-card rows' `1/1`, `1/0` and `1/-` now follow from the rule; the disagreement list is unchanged.
- **F2** (`04-forkaudit-card-arm.md`), fixed as proposed: the `extract.jq` bullet gives the card arm its own span window, a span counting as reached when read after the card-pack call that lists it and before the next edit tool use, and sets `spans_strict_first` and `spans_file_first` to null.
  The `report.jq` bullet says the two first-work-turn rows yield null for such a fork, so they leave the card arm out, and that the header comment states the window.
  I added one detail the fix key left open: `[null, nspans]` is a non-null pair, so `ratio_cell` would not drop it by itself, and the bullet makes the rows yield null for that case.
  The first fork-card cross-check row also wants `strict in first turn` `-`, which pins the null.
- **F3** (`03-per-card-layout.md`, `02-card-pack-verb.md`), fixed by the recommended route, with one refinement: `packLayoutOf` takes `geom Geometry` and returns `PackLayoutCard` only when `AnchorRoot` equals `WorktreeRoot`.
  A standalone geometry falls through to today's `kickstart_pack_file` choice instead of always rendering the file layout, so a standalone run keeps exactly the layout it renders today.
  Under the default config that is the file layout, as the fix key says.
  `buildKickstartPack`, `BeginBatch`, `Uses:`, the doc.go sentence and the ImpactSummary follow.
  `TestBeginBatch_KickstartPack` gains a `standaloneGeom` field and the row "a standalone geometry keeps the file layout under per-card", which wants the pack file and `pack_layout=file`.
  Card 2's degradation paragraph now says a standalone run never gets the per-card prompt.
- **F4** (`02-card-pack-verb.md`), fixed as proposed: the empty-card row splits into a `go` plan row (plain path plus `plan:` handle, `#### Unresolved` naming the handle, `card=1 resolved=0 unresolved=1`) and a `language: none` row (plain path only, no `#### Unresolved` heading, `resolved=0 unresolved=0`).
- **F5** (`02-card-pack-verb.md`), fixed as proposed: `contracts/stencils#WebsterFragmentPackCard` joins the Edit list, and the Docs paragraph says its doc comment also names rendering one card alone as `lyx loom card-pack`'s output.
  `lyx loom validate-plan` passes, with informational findings only.

## Disputed

None.

## Deferred

None.
