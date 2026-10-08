# Card 17 — llm-hook-capture

**Edit:**
- `internal/shuttlecli/smoke_turns_test.go`

**Uses:**
- `plan:internal/shuttleengine#ReadSessionStates`
- `plan:internal/shuttleengine#SessionFold`

**Intent:**
Add an `llm`-tier scenario to the turn smoke suite that captures real `UserPromptSubmit`, `Notification` and `SessionEnd` payloads, pinning card 3's parse of them against what Claude Code writes.

The scenario starts an interactive run on the existing `turnHub` substrate through `llmkit`, whose prompt asks the agent to reply one line without writing its output file, so the run holds.
It waits for the events file to carry a `Notification` whose `notification_type` is `idle_prompt` (the only notification type a bypass-mode session reaches), bounded by `turnScenarioTimeout`, and checks that `ReadSessionStates` lists the run as `asking` (interactive, no output); it then sends `/exit` so the session ends, and waits for the `SessionEnd` line.
It asserts that the events file holds a stamped `UserPromptSubmit`, a stamped Stop, a stamped `Notification` `idle_prompt` and a stamped `SessionEnd`, each stamp naming its hook with a parseable time; that `ParseSessionSignals` reads them as turn start, turn end, idle notice and a process-ending session end; and that folding those signals in one `SessionFold` (interactive, outputs missing, liveness alive) reads the history `busy` `turn`, `asking`, then `dead` with the session end's reason.
The fold is read directly rather than through `ReadSessionStates`, because the run's record stops reading `running` once its pane is gone.
Like its siblings, a failing scenario keeps its events file and transcript in a logged temp directory for promotion to a corpus case.
The scenario's own payloads are its fixture; it reads no other worktree's files.

**ImpactSummary:** Adds one llm-tier scenario to the shuttle turn smoke suite; no production code changes.
