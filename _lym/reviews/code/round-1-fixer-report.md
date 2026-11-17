# Round 1 fixer-report

## Changes

- F1 (LOW), fixed as proposed: `dirtyOwnedFile` in `internal/gatecli/testmaptree.go` reads `Fabric.CodeWorktreeChanges()`, as `dirtyModuleFile` does.
  Added the subtest `ADirtyRecordsFileLeavesAPackageMeasured` to `TestGateTestMapLinkedPackages`:
  a records-side `caller/records-note.txt` leaves `caller` built.
  Commit `fix F1`.
- F2 (LOW), fixed as proposed: `internal/gatecli/cli.go` help text restores the enclosing-slot and summary-line sentences and the `--gate finalize` and `--upto 2` examples.
  Commit `fix F2`.
- F3 (NIT), fixed as proposed: removed `--tags` from the `--map` refusal list and message in `run.go`, the `--tags` row in `cli_test.go`, and the mentions in the `cli.go` help line and `doc.go` Coverage maps section.
  The `cli.go` hunk of this change sits in the F2 commit, since both edit the same help text; the rest is in commit `fix F3`.
- Self-check finding, fixed: `doc.go` still named `Fabric.Status` for the dirty-file read; it now names `Fabric.CodeWorktreeChanges` and says only the code side counts.

- Gate finding (`TestCLIForm_EveryCommand`: `lym gate test` Long was 2213 bytes, cap 2000), fixed differently from F2's proposal:
  the F2 restoration pushed the help text over the cap, so `cli.go` now keeps the enclosing-slot sentence and drops the summary-line sentence,
  drops the restored `--gate finalize` and `--upto 2` examples, and shortens the `--map` line to name no flag list (the package doc holds it).
  Not made: restoring the summary-line sentence and both examples, since together with the `--map` text they exceed the cap.
  Tier0 passes for `./cmd/lym` and `./internal/gatecli`. Commit `fix gate help cap`.
- Gate finding, fixed: the comment at `internal/gatecli/testmap_integration_test.go:869` reached the line-length limit;
  it is now two sentences on two lines. Commit `fix gate line length`.

`lym gate test --wait ./internal/gatecli` passed at tier0 and at `--tier 1` before the gate finding; the comment-only repair changes no behavior.

## Self-check

- fix-consistency, LOW: `doc.go` named `Fabric.Status` after F1; fixed as proposed.
- fix-code-correctness: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
