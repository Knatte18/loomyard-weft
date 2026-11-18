## Goal

```markdown
Left open by the last code review of `leftover-strands-and-runs` (`3746960e`), at MEDIUM.

`reconcileInterruptedStep` in `internal/shedverbs/loop.go` recognizes an interrupted stop from the status error text.
A shuttle run left running by a step that wrote no interrupted record is therefore not reconciled on a re-step, and the re-step can still end on it.

Idea: reconcile from the run records themselves (a run reading running whose waiting process is gone), not from the error text.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
