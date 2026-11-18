---
round: 1
focus:
  - "internal/shuttleengine/relaunch.go and run.go: when a gate re-prompt finds its strand untracked, does the session resume correctly, and do a live holder or a non-resuming engine end the loop with the original error instead of a hang or a double launch?"
  - "internal/shuttleengine/stepstop.go, wait.go and internal/loomcli/looparm.go: does the re-step reconcile every run of the dead step that reads running with a gone waiting process, without stopping runs of another step or a live run?"
  - "internal/shedverbs/loop.go and loopwait.go: does an interrupted child fill the error form and does the watchdog kill an idle child, and is the cause recorded where the loop exits, as the second part asks?"
  - "Second part, bullet 2: does the diff check that the interrupted-step record names the step actually in flight, consistent with the history, or is that request left unaddressed?"
  - "Doc and comment updates in internal/shuttleengine/doc.go and internal/shedverbs/doc.go: does each claim match the code as the diff leaves it, and is the repository's own doc-update rule met for the changed behavior?"
---

Both parts of the board entry have code in the diff; the focus targets failure paths and the unaddressed bullet.
