---
round: 1
focus:
  - "internal/proc/reap.go and its callers in internal/beamcli/cli.go, internal/orchcli/watch.go: does the startup reap of inherited children wait only on children and never on a pid the process still needs to Wait on itself (a double Wait, a reaped pid reused, a blocking Wait3/Wait4 that hangs a start)?"
  - "Every detached spawn the diff wraps in proc.Reap (boardengine, fabricengine, fronttask, reedengine, reedcli, battencli, adoptcli, termwindow, dobbyengine): is each Start error path handled so no Reap goroutine waits on a nil or unstarted process, and does no caller already Wait on the same cmd?"
  - "Spawns the diff does not reach: does any other child lym starts without waiting remain unreaped, given the decision record's goal that dobby and lym orch watch reap every child they start?"
  - "internal/runproc/record.go and end.go: does the registry treat a state-Z pid as gone in every listing path that `lym tender status` and `lym tender reap` use, and does End's group walk still reach a zombie leader's group without signalling a pid it should not?"
  - "Non-Linux behaviour of the new runproc and proc code: is the zombie check guarded so Windows and macOS builds keep their prior behaviour?"
  - "Tests: does proc_linux_test.go and record_test.go cover the new reap and zombie-as-gone behaviours without duplicating each other, and is any new behaviour (End on a zombie leader, refused dobby start reaping no test child) left uncovered?"
  - "PATTERN.md and pattern/PATTERN-spawn-observability.md: do the changes land alongside the code that motivates them, and do the new doc.go lines state the rule without pinning a perishable list of callers?"
  - "The decision record's Scope, Decisions and Acceptance criteria are empty: judge scope against the Goal alone, and check that the commit-message claims (re-exec drops waiter goroutines; each listed spawn is reaped) are borne out by the diff."
---

The answer key is a decision record alone, so the Goal is the measuring stick.
The mechanical gates cover build and tests; the focus entries aim at the places where reaping can go wrong without a failing test.
