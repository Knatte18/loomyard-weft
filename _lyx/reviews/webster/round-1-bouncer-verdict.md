---
verdict: BLOCKING
rationale: "F1-F8 are fixed and verified on HEAD, but new comments in seven places still use fixed-column wraps, which CONSTRAINTS.md's Comment Line-Break Convention forbids for all Go comments, test files included. The report missed six of them and the F2 fix left the seventh (planwrite_test.go:241, :394, singlellm_test.go:977, :1008, entries_planwrite_test.go:162, prepare_test.go:155, loomrecipe/fixture_test.go:74-78, command_test.go:265, planwrite.go:116)"
---

# Webster-Review round 1 — judge verdict

## Range and verification

`product.parent` in `_lyx/shed/plan-write-respawn/status.json` is `main`, so the range is `git diff $(git merge-base main HEAD)..HEAD`: six card commits, two merge commits and the eight fixer commits.
I re-ran `go build ./...` and `go vet -tags smoke ./...`, and ran `go test -tags integration` on every touched package plus `loomrecipe`, `stencilcli` and `loomcli`.
All of them pass on HEAD (`e29b05465`).

## Round report findings

The report is sound, and every finding is fixed on HEAD:

- **F1**: `NewPlanDirRotator` now calls `stalePlanRotation` (no disk writes), renders the block, and only then calls `rotateStalePlanDir`.
  `MissingStencilWithPriorFilesIsAnErrorAndMovesNothing` asserts the plan directory is untouched and that a retry announces the file.
- **F6**: `fork_mode_real_notice_pwsh`, `fork_mode_real_notice_posix` and `TestBuildResumeCmd_NoticePosix` pin the real notice sentences inside the `WithEnv` wrap on both shells, and on the posix resume line.
- **F7**: the rotator now wraps with `loomshed: announce prior plan:`.
- **F2–F5, F8**: the production-file comments they named are now one sentence per line.

The report has no `## Focus departures` section, so there is nothing to ratify or reject.
I agree with its per-card mechanical conclusion.
The plan has no `Rename` or `Delete` group, so the type-specific checks reduce to Edit/Create blast radius.
Grep confirms that `NewPlanDirRotator`'s only production caller is `planWriteEntry`, and that the build compiles every `prepareFreshSpawn` supplier under `func() (string, error)`.

## Why BLOCKING

`CONSTRAINTS.md`'s Comment Line-Break Convention covers Go comments "doc and inline alike" and makes no exception for test files.
The round-1 report applied it only to production files.
These new or changed comments in the diff still wrap at a fixed column or break mid-phrase:

- `internal/loomshed/planwrite_test.go:241`: the `seedPriorPlanStencils` doc breaks "into a temp / stencils directory".
- `internal/loomshed/planwrite_test.go:394`: the inline comment the F1 fix added breaks "where a retry / finds it again".
- `internal/shedadapters/singlellm_test.go:977` and `:1008`: both new test doc comments break mid-phrase ("a / preparation", "a preparation / returns").
- `internal/shedrecipe/entries_planwrite_test.go:162`: the `TestPlanWriteEntry_CallAppendsPriorPlanBlock` doc wraps at a fixed column across three lines.
- `internal/shuttleengine/claudeengine/prepare_test.go:155`: the `TestPrepare_AppendSystemPromptMatchesDenyNotice` doc wraps at a fixed column across three lines.
- `internal/loomrecipe/fixture_test.go:74-78`: the changed `seedBouncerStencils` sentence breaks mid-phrase on its edited lines ("or / Webster-Review", "plus / loom-template-prior-plan", "each seeded / from").
- `internal/shuttleengine/claudeengine/command_test.go:265`: the break after "argument," comes before a participial phrase, which is not an independent-clause boundary.
- `internal/loomshed/planwrite.go:116`: the F2 fix breaks after "prompt:", but the next line ("empty when nothing moved, otherwise …") is a fragment, not an independent clause.

Each finding is LOW, and each fix is a comment-only rewrap.
The rubric names comment-convention compliance as a dimension to flag, and these lines are in the diff, so the gate should not pass them.
