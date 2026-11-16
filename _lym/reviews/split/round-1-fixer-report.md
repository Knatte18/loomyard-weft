# Fixer report: split round 1

Every finding is fixed in `_lym/raddle/proposal.yaml`.
The file still parses as JSON, and each part's package sizes sum to the line count its cost line states.

## Changes

- **F1** (fixed as proposed):
  `machine-registry-holds-catchup` now exports only the one hold write its upgrade first read calls, writing an `adopt` or `adopt-fix` hold, beside the registry and hold types and readers.
  Its `internal/boardengine` size drops from 650 to 450, and its cost line now reads about 2370 lines.
  `adopt-foreground` lists `internal/boardengine` at 250 lines.
  Its delivers build the rewrite, replace-with-adopt-fix, try-count update and clear setters beside their callers, replacing "no boardengine change".
  Its exports name those setters, and its takes drop them for the hold types, readers and the upgrade's hold write.
- **F2** (fixed as proposed):
  `transition-push-takeover` exports the hand-over mark's path and reader without the drop function, and its `internal/fabricengine` size drops from 1350 to 1340.
  `adopt-foreground` takes the mark's path and reader.
  It delivers step 3's takeover write dropping this machine's hand-over marks of each moved slug, through a drop function it adds beside the mark reader.
  Its `internal/fabricengine` size rises from 100 to 120.
  For consistency, `leftover-pairs-teardown`'s takes now name that drop from `adopt-foreground`.
  Its D7 re-adopt criterion rests on the drop.
- **F3** (fixed as proposed):
  `transition-push-takeover` depends on nothing.
  `board-prints-root-beam` depends on `adopt-foreground`, the last part listing `internal/boardengine` after F1's fix.
  It shares no package with `leftover-pairs-teardown`, `hub-stop-all` or `machine-start-launchers`, so it runs beside them.
- **F4** (fixed as proposed):
  `adopt-foreground` delivers one pull-first Bolt write in `boardengine`.
  That write records the `adopt` hold, or under `--take-hold` the rewritten hold, together with the subjects' owner and this machine's registry entry in one commit.
  On a rejected push it re-reads Bolt and refuses on an owner of these subjects or a hold another machine wrote since.
  Its exports name the one-commit adopt write.
  Its takes add the registration write with D3's refusals from `machine-registry-holds-catchup`, plus `boardengine.SetOwners`'s pull-first commit and re-read (`internal/boardengine/ownerset.go`) as the pattern to follow.
  No other part calls the write, so no other part's takes change.
- **F5** (fixed as proposed):
  `adopt-foreground` takes the `fix-tries.json` reader in `internal/adoptcli/holds.go` that the upgrade's first read calls.
  Its delivers keep that reader for the upgrade's first read.
- **F6** (fixed as proposed):
  `machine-start-launchers` delivers the launcher added at the dobby start and removed by every catch-up, the dobby start's included, that finds this machine owns nothing in the hub, and added and removed by the verb.
  Its takes name the catch-up as run at the dobby start, after a sleep and on a hand pass.
- `adopt-foreground`'s package sizes now sum to about 3920 lines, and its cost line says so.
  `transition-push-takeover`'s cost line reads about 3170 lines.

## Disputed

None.

## Deferred

None.

## Rulings

None.
