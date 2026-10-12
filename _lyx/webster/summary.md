# intake-bugs-1012: four intake bugs closed

Batch 01 (`01-external-test-unit-dir`) recorded `done` at head `b6af4a84c801f0c1c7d51e54ee820c01a7dbb3cb`, in one fork pass with no recovery.

The batch landed five cards, one commit each:

- Planparser's target-dir mapping now maps an external-test glyph unit to its package directory, and the glyph chokepoint sanctions that one mapping.
- A new plan check refuses a relative package argument that names no directory in a Verify or `## verify:` command.
- `lyx gate test` refuses such an argument before any slot or prebuild.
- The shed loop stops with `status-missing` or `status-unreadable`, names the status file and loop log, and leaves the file as found.
- The runnable-group audience rule is written into the cli-cobra background, the overview and the Plan-Review rubric.
  The commit body also closes #584 with no code change, citing the bounded retry and two `TestSweep` runs.

## Deviations

`record-batch` listed files changed outside the declared file-ops, all informational: `PATTERN.md`, `docs/overview.md`, the `internal/planparser` files `verifymodule.go`, `validate.go`, `verifypackage.go`, `vocabname.go` and `targetdirs.go`, `internal/gatecli/cli.go` and `run.go`, `internal/shedverbs/loop.go`, `loopenvelope.go` and `spec.go`, and `internal/shedengine/run.go` and `goto.go`.
The fork also reported trimming a `PATTERN-cli-cobra` line in `PATTERN.md` to stay within its size limit, and that `docs/glyph.md` does not exist in this repo.
