---
round: 1
focus:
  - "internal/gitexec/gitexec.go: does the retry fire only when the start itself failed with EAGAIN, never after git ran and exited non-zero or after a context cancel, and are the tries, the doubling backoff and the Warn log as the decision record states?"
  - "internal/proc/spawnagain.go and its users in reedengine/overlay.go and gitexec: is the refusal record safe under concurrent callers, and does reed's retry behave as before apart from using the shared predicate?"
  - "internal/shedtransient and shuttleengine/wait.go and shedadapters/multillm.go: is `ErrAgentDied` wrapped only for a chair `died` outcome that started, and does `host-resources` apply only to an EAGAIN error or to a died agent within the window of a noted refusal, leaving the existing rows and timeouts unchanged?"
  - "internal/shedverbs/loop.go: are the interrupted cause (command plus stderr excerpt, Go crash first lines over trailing stack), the trace-file fallback to captured stderr, the producer read after the child exited with fallback to the one before, and the failed-stop write before the interrupt hook each correct, including the paths where stderr is empty, the status file is unreadable or the child left no envelope?"
  - "internal/shedverbs/loopenvelope.go and internal/sheddriver/driverules.go: is the new pause-reason key optional and tolerated when absent, and does the driver take it only when the envelope's own reason is empty?"
  - "internal/loomcli/looparm.go and internal/loomengine/anomaly.go: does the interrupt hook record the state and error the status file holds after the stop, and does `ClassifyHalt` report `interrupted-stop` for an interrupted halt in any non-blocked state without changing other anomaly classifications or the halt note format?"
  - "Tests: do the new tests inject the retry wait and stub the spawn without a real clock, avoid duplicating existing coverage, and cover each acceptance criterion's behavior?"
  - "Docs and PATTERN: do the changed packages' doc.go files describe the new behavior, and does the diff honor the PATTERN constraints the decision record lists, with any PATTERN.md change the declared changes call for landed?"
---

Seeded from the board entry, the decision record and the changed-file list, without reading the code.
