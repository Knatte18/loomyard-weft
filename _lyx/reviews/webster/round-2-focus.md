---
round: 2
focus:
  - "internal/shuttleengine/send.go (`confirmSubmitted`, `awaitSettledBox`) and internal/shuttleengine/wait.go (`handleGatedBoundary`'s unsent re-prompt): does every retry loop around a real tmux spawn (Enter, capture-pane, re-prompt resend) end by attempt count as well as by elapsed time, so that it terminates under a clock that never advances, as PATTERN-spawn-observability requires?"
  - "internal/websterengine/runlevel.go (`reclaimEntryTimeStrands`), beginbatch.go and recoverbatch.go: does every site that stops a leftover live strand before respawning log a Warn naming the guid and that an in-flight batch restarts from state.json, before the stop, on every path including the ones that stop without erroring?"
  - "internal/shuttleengine/run.go (`sendVerified` failure messages), template.yaml and config.go (send key comments): does each error message and each config comment describe what the code does on that path, including a send whose window closed before anything was typed?"
---

The three entries each name a site and a question; none pre-states an outcome.
