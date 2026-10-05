# Round 4 fixer report

## Changed

- F1: `_lyx/plan/17-orch-role-run-loop.md` run-loop bullet now states that `lyx batten pause` and `lyx batten goto` leave the batten run paused and that `lyx batten run <slug>` resumes it.
  The `orch-template-role.md` claims row gains a must claim for that resume, and `Uses` gains `internal/battencli/cli.go`.
- F2: card 17 `Uses` gains `internal/loomengine#LoomFrictionDir`, and the After landing bullet cites it beside `DriveReportsDir`.
- F3: `_lyx/plan/20-drop-getter-delegation-tests.md` deletes `TestRunner_ContextTokens_Delegates` too, states the one rule (no Runner pass-through keeps a test), and drops `cyclerEngine`'s `tokens` and `known` state; Intent and `ImpactSummary` match.

## Deferred

None.

## Focus verdicts with no finding

- Card 21 startup row: one `elapsed` representation (`atLeastTimeout` or `zero`) covers both causes, and only the silent row sets `SkillLoadTimeout`.
  Not a defect.
- Card 21 watcher row: the full row, every advance, the end turns and the final check are declared, and the fold is stated once with no fallback clause.
  Not a defect.
- Card 17 Escalations: circling, `lyx loom goto` and `lyx shed goto` each name `lyx loom start`, matching `internal/loomcli/circling.go`, `internal/loomcli/cli.go` and `internal/shedcli/cli.go`; the claims row has a must claim for each.
  Not a defect.
- Card 17 After landing: the claims row pins the prime's own copies and the removed `lyx fabric remove`, and both directories are durable (`_lyx`), matching `PATTERN-durable-vs-ephemeral-state`.
