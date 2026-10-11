# Advisor 1 notes — dbench-ohsm

## Modules the task falls under

| Item | Module | Border |
|---|---|---|
| 1 gate shell cut off | `internal/shuttleengine` (`run.go` wait and finalize, `config.go` `background_shell_wait_min`, `EndedShells` record) | `websterengine` reads the run record; `PATTERN-completion-signal` governs a negative "did it finish" answer |
| 2 Master fixes verify failure | `contracts/stencils/webster/` (`webster-template-master.md`, `webster-body-verify-fix.md`) and `internal/websterengine` (`audit.go` parent-write block) | stencils are read from a told directory (`PATTERN-stencil-ownership`) |
| 3, 4 recovery | `internal/websterengine/recoverbatch.go` (`uncommittedPathsBlock`, `loadRunWrites`, `writtenWorktreePaths`), `contracts/stencils/webster/webster-prefix-recovery.md`, `internal/webstercli` for the exit code | attribution is by shuttle run id today; the board asks for attribution by batch |
| 5 Create check | `internal/planglyph/donecheck.go` (`DoneChecks`, `doneCheckVerdicts`), `internal/websterengine/recordbatch.go` (`create-not-done` refusal) | resolution is the external module `github.com/Knatte18/quarry` v0.2.1, so unexported and test-file resolution is not lyx code |
| 6 ambiguous members | same `donecheck.go`, plus `internal/planglyph` `resolve.go` (`glyph-ambiguous`) and `create.go` (`create-already-exists` for ambiguous) for the Plan-time validator | quarry `ResolveResult.Candidates` carry `File`, so lyx can classify build constraints itself |

## PATTERN entries touched

- `PATTERN-refusal-way-forward`: the reworded `create-not-done` refusal and any new recover-batch refusal need a spec row in the same commit and a test that reaches them.
- `PATTERN-no-denied-recovery`: the recovery prompt naming `lyx gate test` and `lyx loom lint-comments` as allowed must not name a command the agents' settings deny.
- `PATTERN-completion-signal` and `PATTERN-shuttle-stop`: item 1 changes how a run with outstanding shells settles; the stop must go through shuttle's stop verb, and `allOutputFilesExist` is what let the run read `done`.
- `PATTERN-friction-capture`: a discarded-shell stop is a halt-like event; check whether it needs a Go-authored friction note.
- `PATTERN-stencil-ownership`, `PATTERN-parent-directive`, `PATTERN-edit-directive`, `PATTERN-producer-pointer-rule`: item 2 and 4 stencil edits.
- `PATTERN-sole-parsers`: the Plan-time validator for item 6 lives in `planparser`/`planglyph`, not a new parser.
- `PATTERN-test-economy`, `PATTERN-test-speed`: extend existing `donecheck_test.go`, `recoverbatch_test.go`; the test budget file `cmd/lyx/testdata/test-budget.yaml` may need listing under `Edit:`.
- `PATTERN-gate-self-check-parity`: applies only if the Plan-time flag becomes a new gate with a CLI self-check verb.
- CLAUDE.md: docs land in the same commit (`doc.go` of `shuttleengine`, `websterengine`, `planglyph`).

## Board entries touching this task

- `dbench-ohsh`, `dbench-ohom`, `dbench-omsm`, `dbench-n0fm`, `dbench-n0oh`, `dbench-n0om`, `dbench-solo` carry the same title as `dbench-ohsm`: sibling benchmark variants of one task, not separate scope.
- `parent-review-hardening`: wait bound off the boundary and no mid-turn sends; likely adjacent to item 1.
- `webster-parallel-execution`, `weaver`: alter who implements a batch; item 3's attribution by batch must not assume one implementer.
- `gate-checklist-pane`, `spawn-audit-log`: adjacent to recording a discarded gate shell.
- `audit-reader-globs`: touches the same fabric audit as item 2.
- `proc-child-lifetime` (264): owns gate-test child lifetime in `internal/proc`; item 6 names `StartTime` in `internal/proc` as a per-GOOS member.

## Conflicts and risks seen

- No PATTERN entry forbids any item. The task's item 1 offers two options (hold the turn end, or stop explicitly and record the discard); only the second keeps `background_shell_wait_min` meaningful, since its doc says it "expires nothing".
- Item 5: the `DoneChecks` doc comment says Create accepts `found` or `multipart` only, and `doc.go` of `planglyph` repeats it; any change to item 6 must update both.
- Item 5 and 6 cannot change quarry from this repo (separate module, v0.2.1); the fix is in lyx's reading of the answer, or a quarry bump as a separate decision.
- Item 5, checked on this tree with `lyx quarry resolve`: `cmd/testtiming#rusageStats` and `internal/testkit/tmuxkit#tmuxAllowed` both answer `found`, so quarry resolves unexported identifiers.
  The unexported-target premise of #511 does not reproduce here; the batches' refusals more likely came from an ambiguous answer (two build-tagged declarations at the time) or from a worktree where the member did not exist yet.
  Still unchecked: members declared only in `_test.go` files.
  The refusal today says only that the target "still does not resolve"; it names neither the looked-up key nor the status, which is the diagnosis gap the board asks to close.
- Item 6, checked: `internal/proc#StartTime` answers `ambiguous` with per-file candidates, and each candidate carries `File`, so a disjointness test can run on file names (GOOS/GOARCH suffix) and `//go:build` lines read with `go/build/constraint`.
