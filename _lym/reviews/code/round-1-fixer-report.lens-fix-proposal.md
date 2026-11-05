```yaml
lens: fix-proposal
outcome: reported
```

No findings.

- **F1 (zero `TurnEndedAt` row):** The proposal holds. After `ackedRing`, `mark.answered` is true. `verbs.go:378` and `verbs.go:656` set it, and the only other writer is `markRingAnswered` at `deliver.go:278`. So `unanswered` is false and the hold inside the window can only come from `woken`.
  - A zero `TurnEndedAt` makes `!idleness.TurnEndedAt.After(mark.at)` true, so `woken` is true and the bell is held. The row returns at least 30 s only because of that branch.
  - Remove the `woken` clause and the bell comes after the 5 s grace. The row fails, so it exercises the branch it names.
  - I found no case where the proposal is wrong.
- **F2 (doc comment line):** It is accurate. It matches the `wakeHookStrand` comment at `deliver.go:256-257`, and no sibling comment contradicts it.

My one tool call: a grep for `markRingAnswered|answered` in `internal/creelengine` non-test sources. My context did not show where `mark.answered` is set, and the check on whether `unanswered` or `woken` causes the hold depends on it.
