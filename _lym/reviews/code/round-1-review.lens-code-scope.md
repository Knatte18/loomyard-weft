```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. Trace each change in the diff to the goal points of the board entry, and check that each goal point is built.

**Decisions (board entry `## Goal`; the record has no `Added after Discussion` entries)**
- G1 Remove the `reach`, `redundancy` and `mutation` report steps, the gate key's `reports` key, and their stencil text: the reach turn, the review markers, and every prompt or rubric line that names a report.
- G2 Remove `lym gate reach`, `gate mutate`, `gate redundant` and `gate test --map`, with their help, docs and PATTERN text.
- G3 Remove the stored maps, their writers, readers and upkeep, and any dobby or row step that starts a map run.
- G4 Remove the tests of everything removed.
- K1 Keep `cmd/testtiming -redundancy` and the part of `internal/testcover` it needs.
- K2 Keep the red-first gate and `internal/testreach`.
- H1 Hub config: the change description names the `lym config reconcile --layer default --apply` step.

**Changes → decision**
- Report chain removed from `gatecli/run.go`, plus deleted `reports.go`, `reach*.go`, `redundancy*.go`, `mutation*.go` → G1
- `GateKey.Reports`, the `Report*` consts, the `reports` lines of `template.yaml`, the `reports` validation in `gateslot/config.go`, and the lenient-load row in `config_test.go` → G1
- The `redundancy_*`, `reach_budget_min` and `mutation_*` keys removed from `Config`, `configFile`, `ConfigSpec`, `keynames.go` and the template comments → G1/G2 (these keys bounded only the removed steps and verbs)
- The processcap and memoryguard carve-out (`runningCommands`, `foreign`, `countable`, `commands` on `testRequest`, tierrun registration) → G1, forced: it existed only for report steps beside a tier
- Commit 3026c4337 (`guardOwnTests`, `startMemoryGuard`, `onKill`, `enclosing`; the allowlist entries in `testonlycode_linux_test.go`) → G2, forced: their only production callers were the removed verbs
- The reach turn (`verifytree/reach.go`, `shedrecipe` `VerifyReach` and `entries_gate.go`, weaverengine `ReachPending` and `strand.go`, the loomcli wiring, the envkit `nilLegal` rows) → G1
- `RerunMark` and `GateEntry.Rerun`, plus the gatefixer `rowgate.go` and doc edits → G1, forced: the mark served only the reach turn
- The rubric marker sections, the `shedadapters` rubric and bouncer notes, the shedrecipe `Env` and bouncer/burler notes, the loomcli note files, and `rubric_test.go` → G1
- `--map`, the `redundant`/`mutate`/`reach` commands, `testmap*.go`, `cli.go`, `detachfront.go`, the `flagShapeRefusal` branch, and the gatecli `doc.go` sections → G2
- PATTERN.md, `PATTERN-coverage-map-reader` and its test `cmd/lym/coveragemapreader_test.go`, `PATTERN-test-speed`, `PATTERN-gate-slot`, `PATTERN-daemon-wakeups`, `docs/overview.md`, the README feature bullet, and the gateslot doc → G2/G3
- The README research row "Declined; the maps were removed" → forced (the row's status became stale through G3)
- testcover cut (blockkey, coveragemap, mutant*, linemutant, sourcescan, testfuncs, packagedirs, `ProfileByPackage`, `JudgeAgainst`/`JudgeMap`/`CoveringTests`, the `Toolchain` fields `Dir`, `Ctx`, `BuildTimeout` and `GuardKilled`, the SpawnVerdict JSON tags) → G3 and K1
- testreach cut (reader, index, mapread, fresh, `ChangedFuncs`, `ChangedLines`) → G3
- `ChangedTests` now compares declaration text directly, `Module`/`FileSet` moved, the new two-file row, and commit c28471be9 → K2, forced by dropping the testcover dependency
- `gatecli/gomodule.go` (moved and renamed helpers), `verbDiffBase` and `noteScoper` moved into `redfirst.go`, the `redfirst_integration_test` fixtures rewritten off the testmap seeders → K2, forced by the deletion of the files they lived in
- `lymdirs.TestCoverDirName` removed → G3
- Test edits (`cli_test`, `detachfront_test`, `processcap_test`, `memoryguard_test`, `tierrun_test`, `scopes_integration_test`, `test_integration_test`, `rowgate_test`, `landingdeps_verify_test`, commit 2727aa65d's `wiring_darn_test`, `wiring_test`, `entries_*_test`, `bouncerprompt_test`, `rubric_test`, `verdict_test`, `runner_test`, `testmain_test`) → G4, or forced by the G1/G2 removals
- The change description's `## Hub config` section → H1

**Decisions → changes**
- G1: built
- G2: built
- G3: writers, readers, upkeep and the reach refresh row step are built. The maps already written to disk are not removed (see F1).
- G4: built
- K1: built (`cmd/testtiming/redundancy.go` uses only `Toolchain{Go,Env}` / `{Env,TestTimeout,MemoryLimit,Grace}`, `ListTests`, `BuildBinary`, `RunAlone`, `ScanSpawns`, `ScanKeeps`, `CoveredPositions`, `PackageBlocks` and `Judge`, all kept)
- K2: built
- H1: built, in `_lym/landing/summary.md` `## Hub config`

No change traces to nothing: every hunk is a goal point or forced by one, and no `Pattern-Departure:` trailer is needed.

**Findings**

- **F1 — LOW, scope, location `_lym/landing/summary.md` (`## Hub config`).** G3 asks to remove "the stored maps", but the maps already written stay on disk and nothing names how to delete them. The hub's shared maps under the board directory are still there: `/home/knatte/Code/loomyard-LYMHUB/_bolt/.lym/testcover`, 56M by `du -sh`. Each task worktree's own maps under its `.lym/testcover` also stay. The diff deletes `lymdirs.TestCoverDirName` (`internal/lymdirs/dirs.go`, removed hunk), every reader and writer, and adds no cleanup: `internal/cleanupcli` has no testcover hit. The change description names no manual step either. Fix: add a line to `## Hub config` naming the manual removal of the board directory's `.lym/testcover`, and of each worktree's own. Or have the hub's routine cleanup remove the directory.

Tool calls:
1. `git show --stat` of c28471be9, 2727aa65d and 3026c4337: my context did not show which files each fix commit touched, which I needed to trace them.
