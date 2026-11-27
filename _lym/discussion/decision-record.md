## Goal

```markdown
Imported from [issue #293](https://github.com/Knatte18/loomyard/issues/293).

## What happened

The task's decision record lists the records push of `webster reset` among the pushes that stay synchronous, because its owed record holds the run at its transition.
The code differs: `fabricSync` in `webstercli` only commits through `Fabric.CommitWithTrailers`, whose spawn starts the pusher.
The pusher writes no record for an owed push.
No card targeted that path, so it was not changed.
The pattern documents `pattern/PATTERN-pair-remote.md` and `pattern/PATTERN-run-fsm.md` describe the code as it is.

## Why it is abnormal

The approved decision and the delivered behavior disagree: a failed push of the reset's records commit does not hold the run.
No card covered this item of the decision record, and the gap surfaced only during documentation.

## What to do

Decide whether the decision record or the code is right.
If the record is right, make the reset's records push synchronous and recorded, and update the two pattern documents.
Also consider checking at plan review that every decision item is covered by a card.

Provenance: task `push-retry-not-halt`, note `20-dobby-push-summary.md`.

## Orch decision

The decision record is right: make the records push of `webster reset` synchronous and recorded, so an owed push holds the run at its transition, and update `pattern/PATTERN-pair-remote.md` and `pattern/PATTERN-run-fsm.md` to match.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-27): No bug fixed

Decision: Nothing changes: 3fb0787e3 on main already pushes the records of webster reset synchronously through pushPairRecords and pairteardown.PushRun, records an owed push, and both pattern documents describe it.

Rationale: Issue 293 describes an intermediate state of push-retry-not-halt; the landed code matches the decision record, and the orch checked reset.go and PATTERN-run-fsm.md. The plan-review card-coverage idea belongs to review-fix-claims-checked.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
