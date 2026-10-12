# Fixer report — plan, round 1

Every finding's premise held against the code; all six are fixed as proposed.

## Changes

- **F1** (`_lyx/plan/16-batten-cleanup.md`): step 3 gains a branch after the absent-but-tracked one: absent on disk, with `Committed` finding `shedrun.SeedRel` but not `shedrun.StatusRel`, skips and reports the slug with the reason "tracked run directory with no committed status", leaves its deletion uncommitted and never empties its scratch by the orphan rule.
  `internal/shedrun#SeedRel` is added to Uses, and `cleanup_test.go`'s table gains a row for the case, with no commit and its aged scratch kept.
- **F2** (`_lyx/plan/16-batten-cleanup.md`): `cleanup_integration_test.go` builds the hub with a task pair (`hubforge.AddPair`) and first runs `RunCLIIn(<task pair worktree>, out, "cleanup", "--apply")`, asserting an error envelope naming the prime worktree, with the done run directory still present on disk and at the records HEAD.
- **F3** (`_lyx/plan/06-testtiming-run-directory.md`): the Edit bullets carry the re-sign arrows `func measurePackageResources(tags, importPath string, env []string, parallel int) (resourceRow, error)` and `func leftoverProcesses(marker string) []string`, matching the current signatures at `cmd/testtiming/resources.go:155` and `:405` minus the dropped directory.
- **F4** (`_lyx/plan/02-tmuxkit-run-directory.md`): the Intent says `TestCheckSockets`' afterRun subtest (`tmuxkit_test.go:294`) passes a `remove` that deletes its directory, and the ImpactSummary names afterRun's three callers.
- **F5** (`_lyx/plan/20-cleanup-verb.md`): the `board` segment opens the board with `boardengine.OpenHub(location.HubPath)` before `Board.Prune(apply)`, and the `scratch` segment reads the same board's `ListTasksFull`.
  Uses replaces `internal/boardengine#LoadConfig` and `internal/fabricengine#BoardDir` with `internal/boardengine#OpenHub`.
- **F6** (`_lyx/plan/20-cleanup-verb.md`): `cleanup_integration_test.go` writes the prime's `logger.yaml` with a short `trace_retention_days` bound and seeds one trace group older than that bound and younger than the default (`logger.DefaultRetentionBounds`, 14 days), asserting the dry run lists it under the `logs` segment and `--apply` removes it.

## Disputed

None.

## Deferred

None.
