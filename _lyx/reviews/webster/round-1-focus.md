---
round: 1
focus:
  - "Cards 2 and 3 (GitHub transport): does a paused call's deadline move by exactly the held time reported by `pauserecord.Network`, and does every multi-request caller derive a fresh deadline per request with no shared deadline left behind?"
  - "Card 4 (`Runner.SendConditioned`): does it refuse at once when the condition (idle-stalled, cause `api-error`) does not hold, never clear the session, and bound its turn-start wait?"
  - "Cards 5 and 6 (orch notice and watcher re-exec): is the notice de-duplicated at queue time and dropped at delivery once its state no longer reads, and does the re-exec happen only at a restart-safe tick with the watcher staying the orch pane's only writer?"
  - "Cards 8 to 10 (tender command, status, revive): does revive run under the pause record lock, and are the process reading, session enumeration, conditioned send and notice queue all seams the engine is told (`PATTERN-told-geometry`)?"
  - "Card 11 (`reed attach`): does it exec tmux in place on Unix only, with the Windows path unchanged and the shortcut guard keeping attach last?"
  - "Card 12 (rename): was it done with `git mv` first and surgical edits only, and do the Rename group's mechanical checks cover every old identifier, in docs and tests as well as code?"
  - "Test economy: do the new tests duplicate existing coverage, and does each behavior the diff adds have a test, per `PATTERN-test-economy` and `cmd/lyx/testdata/test-budget.yaml`?"
  - "Docs in the same commit: do the changed modules' `doc.go`, `docs/overview.md` and `PATTERN.md` match the new behavior?"
---

Focus set from the plan's Card Index and Shared Decisions before any round was judged.
