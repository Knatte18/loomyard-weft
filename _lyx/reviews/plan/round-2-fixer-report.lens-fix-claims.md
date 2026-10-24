```yaml
lens: fix-claims
outcome: reported
```

Task: check that the claims the fix added or rewrote in cards 35, 8 and 2 hold against the code at HEAD.

no findings

- Card 2 and card 8 now each have a row where the records worktree is removed and the refusal names `lyx fabric reconcile`. This holds at HEAD:
  - `ReadOriginFor` (`internal/fabricengine/origin.go:117`) returns not-found without error when the record path is gone. That happens when the whole weft worktree has been removed, so the row reaches the missing-record refusal.
  - Telling this case apart from a deleted record is the cards' own planned design and does not depend on any existing code.
  - Reconcile recreates a missing weft worktree from its branch (`internal/fabricengine/reconcile.go:4`, `ReconcileActionWeftRecreated`), so naming `lyx fabric reconcile` as the way forward is correct.
- The claim "with no commit made" holds for that row, since nothing can be committed on a records worktree that no longer exists.
- In card 35 and card 8, the root pair's way forward is `lyx board upsert` and then `lyx beam create`, along with the assertion that "a root pair cannot become a beam". These are planned-design claims, not facts about existing code, and they match the never-started refusal's route and the rest of card 35.

Tool calls:
- Grepped for `ReadOriginFor` and reconcile's records-worktree handling, because the context did not show the existing code that the reconcile claim relies on.
- Read `ReadOriginFor`'s body and grepped for reconcile's recreate logic, to confirm how a removed worktree is reported and that reconcile restores it.
