---
round: 2
focus:
  - "internal/loomshed/redfirstgate.go, redFirstGate.arrival: when the wait loop fails to send the hold line, is the line offered again on a later pending answer until delivered, with no attempt spent and the check run once per tree?"
  - "internal/loomcli/darnwiring.go, latestDarnOutcome: when the latest Darn-Write history entry is a Stuck that wrote no findings (scope refusal, stop report, session Stuck, invalid decision record, terminal gate reason) after an earlier halt left a kept findings file, does the prior-work note still name that stale file?"
  - "internal/loomshed/redfirstgate.go, redFirstHoldSendFailed and the gateslot no-test-seam title constant: is the record title spelled once, as a constant, everywhere the gate's messages use it?"
  - "internal/loomshed/redfirstgate.go, redFirstGate.final: at a deadline or liveness finalize on an unchanged held HEAD tree, does Final run the red-first check again?"
  - "Comments naming the kept gate findings file (shedrun GateFindingsFile, the shedadapters burlerreview gateFindingsPath field, loomshed doc.go prior-work paragraph, DarnOutcome.FindingsPath): do they match the behavior that a passing call no longer removes the file?"
  - "internal/loomshed/redfirstgate_test.go and internal/loomcli/wiring_darn_test.go: do tests fail when the hold re-arms after a non-hold verdict, when Final meets an exempt entry, when a timed-out exit 4 or 5 run is judged, and when wireDarn's PreflightStencils wrap is reverted?"
---

Settle each question against the diff as it stands.
