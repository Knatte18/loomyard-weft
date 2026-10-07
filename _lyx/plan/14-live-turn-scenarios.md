# Card 14 — live-turn-scenarios

**Create:**
- `internal/shuttlecli/smoke_turns_test.go`

**Uses:**
- `internal/shuttlecli/smoke_interrupt_test.go`
- `internal/shuttlecli/smoke_run_test.go`
- `internal/testkit/llmkit/llmkit.go`

**Intent:**
D10, the live half: `llm`-tier scenarios (`//go:build llm`, reaching Claude through `llmkit.Claude`) that provoke the hard turn-end cases against a real Claude in a real tmux pane, beside the existing shuttle smoke tests and reusing their hub, reed and model helpers.
They are compiled by the plan's `go vet -tags llm ./...` and run only by hand (`PATTERN-test-speed`).

Each scenario drives `shuttleengine.Runner` directly, as `smoke_interrupt_test.go` does, with a recording notifier set through `SetNotifier`, and reads the run's events file through `claudeengine.New().ParseEvents` while `Wait` runs:

- a long background shell (a backgrounded `sleep`), whose turn end reads waiting with that shell, reported by the payload signal, until it finishes;
- a background shell finishing mid-turn, the #411 shape, whose Stop with an empty `background_tasks` reads done on the tick it is read;
- a Monitor, listed in `background_tasks` while it runs;
- a fork (`Spec.ForkSubagents`), listed in `background_tasks` as a subagent while it works, the turn end reading waiting until it returns;
- a message sent mid-turn with `Run.Send`, after which the run still ends done;
- a skill load (`Spec.Skills`), whose load turn end is not read as the run's own;
- a compact of the live session mid-run, after which the run still ends done;
- a turn end without the output file, which holds and notifies exactly once, and then ends done after an answer ending in `MessageTail`.

Every scenario whose agent has background work running asserts that the Stop payload's `background_tasks`, as `ParseEvents` reports it with the payload signal, lists exactly what is running, which is the live check on D2's bound; the question and interrupt cases are the existing `smoke_guardrail_test.go` and `smoke_interrupt_test.go` tests, which card 11 moved to the new reading.
A failing scenario copies its events file and its transcript into a directory under the OS temp dir that it prints with `t.Log` and leaves in place, so promoting it to a corpus case (card 13) is a copy and a trim; a passing scenario leaves nothing behind.

**Verify:** go vet -tags llm ./internal/shuttlecli/...
