# Card 15 — parts-rotators

**Create:**
- `internal/loomshed#NewPartsDirRotator`
- `internal/loomshed#NewPartsPlanRotator`
- `internal/loomshed#archivePartsEntries`
- `internal/loomshed#TestPartsRotators`

**Uses:**
- `internal/loomshed#NewPlanDirRotator`
- `internal/loomshed#TestNewPlanDirRotator`
- `internal/partsparser#Dir`
- `internal/partsparser#File`
- `internal/partsparser#ArchivePrefix`
- `internal/loomshed/doc.go`

**Intent:** Add the fresh-spawn preparations of the two new rows (D4, D6), so an earlier attempt's parts file, part cards and join record never read as a new session's output.
Each runs as the producer's `prepareFreshSpawn`, after the seat probe found no live chair, as `NewPlanDirRotator` does.

`archivePartsEntries` moves every top-level entry of `partsparser.Dir(anchorPath)` whose name does not begin with `partsparser.ArchivePrefix` and that keep does not keep into a new directory `<ArchivePrefix><stamp>[-N]` under it, the stamp the compact UTC layout `archiveTimestampFormat` formats and `-N` the first free same-second suffix; earlier archives stay in place, so archives never nest.
An absent parts directory, or one with nothing to move, is a no-op that creates nothing.

- `NewPartsDirRotator` is `Parts-Write`'s preparation: it archives every entry (keep is never true) and returns an empty amendment.
- `NewPartsPlanRotator` is `Parts-Plan-Write`'s: it first archives every part directory and `join.md`, keeping the parts file (`partsparser.File`'s base name), then runs `NewPlanDirRotator(anchorPath, stencilsDir, archiveWebster, now)` and returns its amendment, which goes to the chair as the prior-plan block.
  The parts archive goes first so a retry never loses that block: a failed parts archive leaves the plan in place, and a failed plan rotation leaves it in place too, while the retry's parts archive has nothing left to move, so the retried plan rotation still announces the prior plan.

`TestPartsRotators` (tier0, a temp anchor with a fixed clock) covers: `Parts-Write`'s rotation moves the parts file, two part directories and `join.md` into one new archive and leaves an earlier archive in place, unnested; a second rotation in the same second takes the `-1` suffix; `Parts-Plan-Write`'s rotation moves the part directories and `join.md` but keeps the parts file, moves the stale plan files and returns the prior-plan amendment; a failing `archiveWebster` after the parts archive leaves the plan files in place and a retry returns the prior-plan amendment; and an absent parts directory is a no-op for both.
