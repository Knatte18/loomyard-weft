# Card 8 — fork-ask-live-scenario

**Create:**
- `internal/shuttlecli#TestSmokeTurnsForkPermissionAsk`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
The live check behind card 7: a fork whose command Claude Code prompts on in bypass mode records an ask signal in the run's events file, which is the signal card 7's latch reads.

`TestSmokeTurnsForkPermissionAsk` is an llm-tier scenario in `internal/shuttlecli/smoke_turns_test.go`, built like `TestSmokeTurnsFork` on `newTurnHub` and `startTurnScenario`: an autonomous run with `ForkSubagents` set, whose prompt has the agent launch one fork, subagent_type fork, whose whole task is to run one Bash command and reply OK.
The command is `rm -f ./smoke-ask-$$`: a computed-path `rm` through a special parameter card 4 lets through, which the decision record assumes Claude Code still prompts on in bypass mode.
The scenario waits through `awaitSignal` for a `SessionSignalAsk` and asserts that at least one ask signal in the events file is not an event, so it came from a permission or input-dialog hook and not the question tool.
The run is torn down by the scenario's existing cleanup, since the dialog is never answered.
When no ask is recorded, the scenario fails and keeps its events file and transcript as the file's other scenarios do, and it is never retried with another command; the decision record's open risk names that outcome, and it means card 7's latch needs another trigger.
The file's header comment names the new scenario among the hard cases it provokes.

`cmd/lyx/testdata/test-budget.yaml` raises `internal/shuttlecli` by one for the new test.
The plan-level verify compiles it under `go vet -tags llm`; the card's own `Verify` runs it once live.

**ImpactSummary:** Adds one llm-tier scenario that only an explicit tagged run executes, and raises the shuttlecli budget row by one.

**Verify:** lyx gate test --tags llm ./internal/shuttlecli -- -run TestSmokeTurnsForkPermissionAsk -count=1
