---
round: 3
focus:
  - "Every way the record names to go on without regression after a Drift-Verify or darn Drift-Baseline halt: check each against shedengine.CheckGotoTarget in internal/shedengine/goto.go, which refuses a target past the run's current row, and settle whether the named verb sequence can actually lift the halt."
  - "The capture child's lifetime on Windows: the loop starts each step through proc.StartTied in internal/shedverbs/loopchild.go, and proc.Detach in internal/proc/proc_windows.go sets no breakaway flag. Settle whether the record's capture survives its step and whether pairteardown and adopt can end it."
  - "The loop's step idle watchdog (step_idle_timeout_min in internal/loomengine/template.yaml, agentActivityFor in internal/loomcli/looparm.go): settle whether Drift-Await's wait, darn's foreground capture and Drift-Verify's mechanical compares count as activity, against the record's claim of no bound of its own."
  - "Under scope full, whether any caller that is not the parent (the arbiter included) can run lyx drift accept at a Drift-Verify halt and resume to Done, and under scope partial whether a test the parent withdrew can be re-accepted at a halt without a DecisionHaltPrefix reason."
  - "Consistency between D2, the acceptance criteria and the parent's Added after Discussion ruling that Drift-Await carries entry_gate drift.enabled and records Skipped with the flag off, including the case of the flag turned off while a capture runs."
---

Focus targets the mechanics that round 2 found broken against existing code, restated so a reviewer can check them without a prior review.
