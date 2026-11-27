# Close a task's scope at Code-Review, and check earlier what PRs find

A task's scope can no longer change once its code is under review.
From the recipe's first Code-Review row until its Finalize row, the `loom decision add` and `loom decision replace` verbs refuse every caller.
A Code-Review decision finding that the record already asks for now reaches a fixer.
Discussion, the plan, Code-Review and the darn writer each check earlier something a reviewer used to catch only at the pull request.

## The scope line

- `loomrecipe.InClosedScope` decides the line from the recipe's own rows.
  The span starts at the first row of the Code-Review segment and ends before the Finalize row.
  A recipe with no Code-Review segment, such as arras, has no line.
  `CodeReviewRunSubdir` returns that segment's run directory.
- `decision add` and `replace` check the line after their raddle check.
  A run whose Code-Review run directory already holds a round counts as past the line, whatever row it stands at.
  So a `goto` back to the writer after a Drift-Recheck stop keeps the scope closed.
- The refusal names the way forward: file the call as a note for a new task with `board upsert` and kind note.
  At a Code-Review decision escalation, the way forward is the in-scope ruling below.
- Finalize stays outside the line.
  When its fixer finds a merge conflict with a recorded decision, the escalation still records the deciding call with `decision add`.
  Feedback the operator gives on the pull request needs no entry in the record.
- The parent's role procedure, the parent directive, the arbiter stencil and its repair-verb text, and `docs/overview.md` state the line.
  The parent's Code-Review review fork no longer gets `decision add`.

## A decision finding ruled in scope reaches a fixer

- `loom circling continue <slug> --in-scope <finding-id>`, with the flag repeated per finding, answers a decision escalation.
  Each id must name a decision-class finding of the escalated round's review.
  The verb refuses any other cause, and any id that names no such finding.
  The ids are recorded as `in_scope` in the round's circling decision file.
- `BuildRoundContext` marks those held findings `InScope` on the next round's prior findings.
  The reviewer prompt marks each one as ruled in scope.
- The round check lets the next review carry an in-scope finding with a class other than `decision`, so that round's fixer takes it.
  It refuses a carry that keeps class `decision`.
  The review may also resolve the finding without quoting an added entry.
  A held finding not ruled in scope keeps its class, as before.
- A new Bouncer key, `scope_closed`, is set on the Code-Review Bouncer row of the tapestry, darn and sampler recipes.
  On those rows the decision escalation stops saying to record each answer with `decision add`.
  Its notice, reason and brief say instead to rule a finding in scope or to file a note for a new task.
  The decision fragment gets a Go-filled `answer_route` marker for this.

## Earlier checks

- Discussion: `## Acceptance criteria` names each figure the change must measure and state, and the case it is measured on.
  It also names each doc statement the change must make true, and each new test's intent and tier.
  The Discussion writer settles these, and the Discussion review flags one that is left out.
  The Describe review reads these criteria as things the description must state.
- Plan: a new `gate.yaml` key, `plan_tier0_budget_sec`, is the most a plan's new and changed tier0 tests may add to one tier0 run.
  It defaults to 60 in the template, and 0 turns it off.
  `Config.TiersValue` ends the tier lines with it, so the planner and the plan review both see it.
  A card that creates or widens a tier0 test states the seconds the test adds.
  The plan review flags a missing estimate, or estimates whose sum passes the budget.
- Code-Review: the rubric flags a figure that a doc, comment, spec or stencil quotes as measured, when the diff changes the case it was measured on and does not re-measure or drop it.
  The darn writer is told to re-measure such a figure.

## Design decisions

- Running the mutation or redundancy step on every C# change is left out.
  Those steps run on Go coverage maps, which are being removed, and no dotnet version of them exists.
- The 60-second default for `plan_tier0_budget_sec` is a starting value, not a measured one.

## Tests

- `TestInClosedScope` and `TestCodeReviewRunSubdir` cover the line's rows in the darn, tapestry and arras recipes, including a run sent back after a started round.
- `TestDecisionCalls_RefusedOnceTheScopeCloses` checks that `add` and `replace` refuse the parent and the operator past the line, and refuse when the read fails.
  In every case the record, its board copy, commits and notices are left alone.
- `TestRecordInScopeContinue` covers the decision file and each of its refusals.
  New rows in `TestBuildRoundContext`, `TestCheckReviewRound` and `TestCirclingVerb_*` cover the ruling's path into the next round.
- A new subtest of `TestBouncer_DecisionEscalation` checks the wording of a closed-scope escalation.
  New rows in the `gateslot` config tests cover the budget line and the refusal of a negative value.

## Merge danger

**Door:** two-way
**Blast radius:** loom
