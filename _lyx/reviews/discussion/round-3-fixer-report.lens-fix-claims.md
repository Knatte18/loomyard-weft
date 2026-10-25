```yaml
lens: fix-claims
outcome: reported
```

Checked the claims the fix added or rewrote against the code at HEAD.

Claims checked:
- D2: a halt leaves `EnteredRow` set and goto sets it to its target. This holds: `goto.go:123`, and `run.go:887` clears it only on `clearEntered`.
- D4: `proc.HoldNamedJob`, `shedrun.LoopJobName` and `proc.TerminateNamedJob` exist. This holds: `proc/tied.go:101,107` and `shedrun/paths.go:412`.
- D4: `spawnLoop` is the one non-test `proc.DetachBreakaway` caller and is reached only from the waiter's spawn branch. This holds: `loopwait.go:111,158`.
- D4: the list of who runs the waiter. See finding 2.
- D6: the parent's resume is told by the last history entry "appended at or after that time". See finding 1.

**1. MEDIUM** — D6, the sentence "An entry is the resume from that halt only when the run status's last history entry is this row's own Stuck in that halt, appended at or after that time".
Summary: identical consecutive Stucks at `Drift-Verify` are folded into one history entry rather than appended, and that entry keeps the first halt's time, so the check can wrongly report that the parent never resumed.
Evidence:
- `shedengine/run.go:700-709` (`foldsInto`) folds a Stuck into the previous history entry when both are budget-exempt and their producer, `Output` and `RoutedTo` are equal.
- `status.go:93` makes a Stuck at a row with no `OnStuck` budget-exempt, and D2 gives `Drift-Verify` no `on_stuck`.
- `run.go:680` says "A fold keeps the last element's At ... sets its LastAt to entry.At".
- The case: decision halt H1 at T1, the parent resumes, the acceptances are ruled, a fresh agent accepts again, and the row returns a second halt with the same reason text. The row records the asked acceptances at T3 &gt; T1, but the folded entry's `At` stays T1. So every later entry, the parent's resume included, reads as no resume, halts again and folds again: the parent can never rule.
- Possible fix: compare the folded entry's `LastAt` when it is set, or make each such halt's reason unique, for example by naming the attempt.

**2. LOW** — D4, "No step child runs that waiter: the waiters are the driver's, the arbiter's relaunch line and the operator's, all outside a step".
Summary: the list of waiters is incomplete, and the "outside a step" claim is not shown for the callers it leaves out.
Evidence:
- `battencli/drive.go:197` calls `shedverbs.RunUntilStop` for batten's own driver.
- loomcli's start, resume and run verbs (`loomcli/start.go`, `resume.go`, `run.go`) also reach the `--until-stop` path.
- The sentence is an enumerated-consumer list that is already stale. Naming the group, "every driver and run verb, none of which a step child runs", needs its own check that no step starts batten's driver.

Tool calls:
- Read `shedengine/run.go:870-895`, `status.go:78-110` and grepped for `HoldNamedJob`/`LoopJobName`: the context lacked how `EnteredRow` is cleared, the history entry fields, and whether the named-job helpers exist.
- Grepped the Repeats/LastAt folding in `shedengine`: needed to know whether a halt always appends a history entry.
- Grepped for waiter callers outside the shed packages: needed to test the D4 list of who runs the waiter.
