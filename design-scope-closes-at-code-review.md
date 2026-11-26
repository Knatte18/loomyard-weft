# Close the scope of a task when its first Code-Review round starts, and have lym hold the line

`scope-closes-at-code-review` · Note · enhancement, loom, burler

Imported from [issue #226](https://github.com/Knatte18/loomyard/issues/226).

## Seen (operator, 2026-11-24)

A task's scope can still change late.
In the C# hubs a parent orch recorded decisions that widened a task while the run was at PR-Review, after Code-Review had finished, and the reviewers never judged the diff against the widened record.
In one run Code-Review stopped at its round cap because a decision finding spawns no fixer, so the widening was in the record but never built.

The operator's rule: decisions go in as early as possible, and once the Code-Review perch has begun it is too late.
The last Code-Review round is the last link that can change anything.
What comes up after that becomes a note for a new task, not a change to this one.

## Idea

Close a task's scope when its first Code-Review round starts, and have lym hold the line, so it is not only a rule for the parent orch.

- Before the line, a decision enters the record as an `Added after Discussion` entry, and `code-scope` and the plan lenses read it.
- After the line, no decision enters the record.
  `lym loom decision add` (and `replace`) refuse once the run stands at or past the first Code-Review round, and the refusal names the way forward: file the call as a note for a new task.
  The refusal reaches the parent and the agent the way the other `decision add` refusals do.
- Two things stay outside the line: feedback from a person through the PR agent, and the mechanical fixes Finalize makes to a merge or test failure.
- The orch procedure and the parent's prompt say the same, so the parent files a note instead of trying.
- Read first how `lym loom decision add` decides where a run stands (`internal/loomcli/decision.go` already refuses at some stages), where the Drift rows sit against the Code-Review perch, whether a Drift verdict can send a run back past the line, and how darn, tapestry and arras recipes place their Code-Review perch.
- Related issues: taking the parent out of PR-Review, and reviewing the Describe text and catching the rest of the parent's findings earlier.
