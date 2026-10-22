---
round: 2
focus:
  - "decision-record.md D9: check the changed text against reed's existing pane reap (Engine.Down and EndSessionByName snapshot pane descendants before kill-session and SIGKILL survivors in internal/reedengine/lifecycle.go and overlay.go); settle whether any process killed today now survives, and whether the Bound names it."
  - "decision-record.md D5: check how a run-lock holder waiting at the network gate is read, given that pauserecord.Network writes its marker only after WaitUncovered returns (internal/pauserecord/hold.go) and awaitRunLockNamingHolder names only a step pid (internal/pairteardown/loop.go); settle that the reading rests on state that exists."
  - "decision-record.md D3, D4, D5: check the stated order of runproc.End, the loop end and the run-lock wait in EndSession (internal/pairteardown/teardown.go), given that a StartTied step child dies by parent-death SIGKILL (internal/proc/tied_linux.go); settle that D5's mid-merge guarantee and the PATTERN-pair-teardown text agree with that order."
  - "decision-record.md D3, D4: check whether End reaches remote git children, which gitexec starts in their own process group without a parent-death signal (internal/gitexec/gitexec.go, internal/proc/procgroup_linux.go); settle what happens to a push past the gate after remove --remote."
  - "decision-record.md D4 and its reed down acceptance criterion: check that reed down's readings are listed in the record itself, whether it runs a networked reconcile under a hub pause, and what stopped and not running mean for it."
  - "decision-record.md D7, D8: check that every site where an owned-elsewhere verdict pauses a run, including batten's owner probe (internal/battenshed/innerrun_owner.go), ends the slug's processes; and that D8's orphan rule and shield cannot reap a live pair's process or be held by a stale running batten record."
---

Targets the sites the round-1 review found gating; the remaining findings are local corrections whose fixes need no fresh look.
