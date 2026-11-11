```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fix set agrees with the record's sibling decisions, criteria and cross-references (fix-consistency lens).

1. **LOW.** Changed text: D8's new `--upkeep` bullet ("a hidden flag that makes the call a pass") and D9's new bullet ("A call naming no package and lacking `--upkeep` … runs as a pass").
   Summary: the change set says what a no-package call does with and without the flag, but not what a call does that names packages and also carries `--upkeep`.
   Evidence: D9's hand-call bullet ("A hand call … naming packages keeps today's blocking take, does not yield … rebuilds the named packages whole") and D8's "makes the call a pass" both claim that call. They disagree on yield, slot rule and whole rebuild, and nothing says which one wins or that the flag refuses package arguments.

2. **LOW.** Changed text: D1's new sentence "The plan itself builds nothing, runs no test and holds no gate slot."
   Summary: the fix moved the Current bullet's "no gate slot" clause into a paragraph of its own, but the untouched sentence a few lines down still says it, so D1 now states the plan's slot rule twice.
   Evidence: D1 now holds both "The plan itself builds nothing, runs no test and holds no gate slot." and "This plan step reads file content … through `fabricengine.Fabric.CodeFileAt` and holds no gate slot." The Constraint PATTERN-gate-slot states it a third time ("the plan step builds and runs nothing, so it holds none").

3. **LOW.** Changed text: D8's carve-out definition ("A recorded failure is either D5's failure field or a failure-only map's `BuildFailure` with its `Commit`").
   Summary: D5's own no-requeue rule was left as it was, so it does not reach the failure-only map that D8 now counts as a recorded failure.
   Evidence: D5 still reads "A pass does not queue that package again while its recorded failure names the current HEAD." That sentence sits beside the failure-field case only. The failure-only map sentence that follows it says nothing about requeueing, so D5 and D8 now define "recorded failure" with different reach.

No further tool calls made.
