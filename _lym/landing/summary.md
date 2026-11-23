# Classify a halt on a newer board stencil as build-skew

A run whose driver runs an older lym build than the one that wrote a board stencil halts as `failed`,
because the stencil guard refuses to render the newer copy.
The halt note recorded that stop as `producer-hard-failure`, and the friction reflection turned it into a lym defect issue (#198).
The skew is not a defect: the driver binary is behind the board, and the way forward is a binary to update.
The driver already hands such a run to a fresh driver when a newer build is on disk, so the run recovers and only the false report remains.

This change gives that halt its own anomaly kind, `build-skew`, so the halt note no longer claims a producer failed.

Closes #198

## What changes

- `loomengine.ClassifyHalt` takes a `buildSkew` flag and returns the new `AnomalyBuildSkew` (`build-skew`) for a `failed` state when it is set.
  An interrupted stop and a blocked state keep their kinds, and any other failed state stays `producer-hard-failure`.
- `loomcli`'s `failedHalt` sets the flag when the halt error's chain holds a `stencilstore.NewerBoardCopyError`, and `haltNote` carries it to `writeHaltNote`.
- The friction reflection stencil `friction-template-reflection.md` drops a `build-skew` halt like a halt caused by the task's own code, so no issue is filed for it.
  The note stays on disk, and the operator can still file through `lym selfreport create`.
- The `loomcli` package doc describes the new kind.

## Design decisions

- The kind is set for every failed halt on a newer board stencil, whether or not the driver later hands the run to a fresh build.
  The halt note is written before the driver decides on a hand-over, so narrowing the kind to the hand-over case would need knowledge the note writer lacks.
  A skew halt with no newer build on disk, or with its one hand-over spent, has the same cause: a lym binary to update.
- No pre-step hand-over and no early skew warning were added.
  The existing hand-over already recovers the run, so the task is limited to the false halt report.
- The kind is decided by `errors.As` on the error chain, never by matching the reason text.

## To check by hand

- Code-Review F2 (class decision, low severity, `internal/loomcli/halt.go`):
  whether a skew halt the driver does not hand over should also read as `build-skew`.
  The decision above settles it as yes; confirm you agree.
- Not run: a live run whose driver binary is older than a board stencil.
  The tests cover classification and the written halt note, not the driver's hand-over.

## Merge danger

**Door:** two-way
**Blast radius:** loomcli
