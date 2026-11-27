---
round: 2
focus:
  - "`internal/shuttleengine/run.go` (`start`, `launch`, the predecessor lookup by role key), reached from `internal/shuttlecli/agent.go` and `operatoragent.SessionSpec`: when a second `lym shuttle agent` of the same role starts in a worktree while the first is live, or after it ended, what carry-over block, carried-instruction record and replacement notice does it get, and do the other callers of `StartSession` keep their carry-over?"
  - "`internal/shuttleengine/run.go` strand launch for the operator agent's unawaited spec: after the operator ends the session, what becomes of its pane, its window, its run record and its listing in `lym shuttle state`, and does a reed resume relaunch it? Compare with what `Runner.Open` gives an opened session, on the launch command and the resume command alike."
  - "`contracts/stencils/operatoragent/operatoragent-template-start.md` and `operatoragent.RenderPrompt`: read the rendered prompt both with a recorded parent and with none, and settle whether every instruction in it agrees with an agent the operator talks to in its own pane, which has no report to write."
  - "`internal/shuttlecli/agent.go` before and at `StartSession`: what does a button click show and do when a pause covers the worktree's task slug?"
  - "Tests of `lym shuttle agent` and of `lym shuttle open --role` in `internal/shuttlecli`, and of the agent spec and prompt in `internal/operatoragent`: would a wrong geometry field mapping, dropped language skills, a wrong start call, a broken name normalization or rule-first refusal, or a dropped or swapped parent or PATTERN directive fail a test?"
---
