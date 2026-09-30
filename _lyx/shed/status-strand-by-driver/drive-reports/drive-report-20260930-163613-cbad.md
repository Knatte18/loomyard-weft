# Drive report: status-strand-by-driver

## Outcome

The run stopped at `state: awaiting` after step 17 (`Publish`), with `continue: false`.
The envelope's reason: "pull request created; awaiting review, then run `lyx loom approve` and `lyx loom start`: https://github.com/Knatte18/loomyard/pull/310".
An operator gate, not a failure: nothing was repaired.

- Baseline: `("Preflight", 0)`, state `running`.
- Final: `("Publish", 17)`, state `awaiting`, progress 11/13 (remaining `Finalize`, `Friction-Reflect`).

## Steps

| # | Producer | Outcome | Next | Trace id |
|---|---|---|---|---|
| 1 | Preflight | done | Loom-Preflight | 80466e715570f1e7 |
| 2 | Loom-Preflight | done | Discussion-Write | c8a9f6c8d05cb785 |
| 3 | Discussion-Write | done | Discussion-Bouncer | a85588725062e968 |
| 4 | Discussion-Bouncer | stuck | Discussion-Burler | 21f1e4a667088dea |
| 5 | Discussion-Burler | stuck | Discussion-Bouncer | b523609c3ca32d2d |
| 6 | Discussion-Bouncer | done | Plan-Write | ce0bd458d8bbceb5 |
| 7 | Plan-Write | done | Plan-Bouncer | 15bbf94d8036f4a6 |
| 8 | Plan-Bouncer | stuck | Plan-Burler | f32d0732f23a0d98 |
| 9 | Plan-Burler | stuck | Plan-Bouncer | 9afab0c0368aba9d |
| 10 | Plan-Bouncer | done | Batchifier | c7befe38fa53d694 |
| 11 | Batchifier | done | Webster | f4c8441bcce2db28 |
| 12 | Webster | done | Webster-Bouncer | 07df350ee08ebe9e |
| 13 | Webster-Bouncer | stuck | Webster-Burler | 16c50ab5ff6ef71f |
| 14 | Webster-Burler | stuck | Webster-Bouncer | 439ce60b099b0166 |
| 15 | Webster-Bouncer | done | Describe | a21c561987716d8d |
| 16 | Describe | done | Publish | b088fe283a2402d2 |
| 17 | Publish | awaiting | — | 1729ad227a90941c |

Each review segment took exactly one Burler round.
No error envelopes, no interrupted invocations, no repairs.

## Warnings seen in traces

- Plan-Gate (steps 7 and 9), informational: `create-new-unit/2-step-drops-status-strand` — Create target `internal/loomcli/smoke_drivingsurface_test.go` introduces a new unit.
- stencilstore (steps 15–17), `class=source-ahead`: board copies of `landing-template-describe`, `loom-template-plan`, `webster-template-master` and `webster-template-integration` are older than the worktree source.
  The warning says to run a production deploy (`update-plugins.sh`); that is operator-only and was not run.

## Operator choices

1) Review PR #310 — then run `lyx loom approve` and `lyx loom start` to resume through `Finalize` and `Friction-Reflect`.
2) Run `update-plugins.sh` when appropriate, to clear the stencil drift warnings.

## Locations

- friction_dir: `/home/knatte/Code/loomyard-LYXHUB/status-strand-by-driver/_lyx/loom/friction` (one note: `burler-webster-r1.md`)
- repairs: `/home/knatte/Code/loomyard-LYXHUB/status-strand-by-driver/.lyx/shed/status-strand-by-driver/repairs/` (not created; no repairs made)

## Self-report draft (not filed, autonomous mode)

Title: Smoke-tier bootstrap handover tests fail when run from inside a tmux pane

Label: bug

Body:
`TestSmokeBootstrap_MalformedStatusProceedsToHandoverAndLogsWhy` and `TestSmokeBootstrap_DiedDriverProceedsToHandoverAndLogsWhy` fail when `$TMUX` is set, which is always true for agents running in reed panes.
`lyx loom start` takes the inside-another-tmux branch and returns a hint envelope instead of reaching the attach the tests expect.
The smoke helper `runLoomCLINoFatal` inherits the parent environment.
Fix: strip `TMUX`/`TMUX_PANE` in the helper, or set them explicitly in the handover tests.
Seen in run `status-strand-by-driver` (Webster batch 03 and Webster-Burler round 1 both hit it).
