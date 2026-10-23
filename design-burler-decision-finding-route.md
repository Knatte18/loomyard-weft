# Burler fix loop cannot route a decision-class finding to the parent

`burler-decision-finding-route` · Note · bug, burler, loom

Imported from [issue #853](https://github.com/Knatte18/loomyard/issues/853).

A Burler plan review raised a `decision`-class finding whose fix is to ask the parent to record a decision.
Round 1 raised the same finding and its fixer deferred it.
Round 2 raised it again, and the fixer could only defer it again.

Why this is abnormal: the fixer's prompt forbids any message about the review to another session, and the decision record is outside the fixer's write surface.
The fixer can neither ask for the decision nor record it, so every round re-raises and re-defers the finding.
Nothing in the loop routes a `decision` finding to the parent.

What to do: route `decision`-class findings outside the fix step, for example escalate them to the parent (`ly:orch`) from the Burler round driver, and stop counting them as fixable work for the fixer.

Provenance: task `batten-cancel-verb`, note `burler-plan-r2.md`.
