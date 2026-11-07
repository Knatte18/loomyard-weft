---
verdict: CONTINUE
rationale: "Round 1 carries F1, a BLOCKING design finding citing PATTERN-test-isolation; the rule makes it gating, and no relabel was applied."
---

F1 says `TestWorktreeSeam_Events` installs the process-global worktree seam before it takes its registered-template copies.
It is BLOCKING and class `design`, and it cites an entry in the constraints list, so it cannot be downgraded.
No finding was relabeled, and the review has no `## Focus departures` section to ratify or reject.
The loop continues so the fixer can move the copies before the install.
