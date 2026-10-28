---
round: 2
focus:
  - "internal/shuttleengine/activity.go, internal/loomcli/looparm.go (agentActivityFor) and internal/loomcli/decision.go (liveRunStrands): does a running record whose waiter is gone still count as a live agent anywhere besides the batten idle notice, and do the 'live run' doc comments there and in internal/battenshed/doc.go match the behavior?"
  - "internal/shuttleengine/carryover.go and carryover_test.go: does the test table pin every unfinished outcome the selection accepts (stopped, died, timeout, waiterless running), and the rule that only the newest run of a role key is considered?"
  - "internal/shuttleengine/claudeengine/signals.go (typedInstruction): is a prompt opening with an absolute path kept as an operator instruction while a slash command is still dropped, with a test row for each?"
  - "internal/shuttleengine/carryover.go (operatorInstructions): when the run's prompt never landed, is the first operator instruction kept, or does the function's doc comment state the loss?"
  - "internal/shuttleengine run.go Send path and carry-over start path: are a failed sent-text append not blocking a send, the Warn log of the carry-over, and the carried block surviving a start retry with one notice each covered by a test row?"
---

The entries name where to look; each question is settled from the committed diff alone.
