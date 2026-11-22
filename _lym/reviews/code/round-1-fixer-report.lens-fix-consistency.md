```yaml
lens: fix-consistency
outcome: reported
```

Task: check fix-consistency, meaning whether twins of the changed text elsewhere still agree.

I found two LOW findings and nothing at `BLOCKING` or `MEDIUM`.

- **LOW — `pushCloneSeed` doc comment (`internal/fabricengine/destroy.go`, ~1711-1719).**
  - It says "A ref at the seed's tip is withdrawn first… the failure names what origin took and what was withdrawn" and "A ref origin does not hold at that tip yet keeps the hub".
  - The new F4 path, where origin's board stands at a third tip, is neither. It leaves origin's board alone, removes the hub and names a join.
  - The doc now describes only two of the three outcomes. `withdrawUnansweredSeedPush` documents the third ("left alone").
- **LOW — `doc.go` paragraph (`internal/fabricengine/doc.go`, ~536-541).**
  - It has the same two-outcome wording about a push that ended unanswered, and omits the other-board outcome.
  - `TestCloneSingleRepo_SeedPushKilledBeforeOriginsAnswer`'s doc (`clone_singlerepo_killedpush_integration_test.go`, ~44-47) has the same omission. It still reads "push applied → withdrawn; not applied → hub kept", while the new `BoardMovedToAnotherTipBeforeTheKill` row exercises neither.

Checked and consistent:
- No other test or doc asserts the removed wording ("the seed returns the records branch…", "origin holds its seed at"). The only remaining "another clone seeded origin" matches are the two new refusals in `pushclassified.go`.
- The `seedPushCause` lease case matches the F4 cause, because it wraps `ErrLeaseRejected`.
- `isAncestorOrEqual` keeps its other callers, so nothing is left dangling.
- The "the seed landed and its clone stopped before it set the upstreams" fragment survives the reworded refusal, so the rows that assert it still hold.

Tool calls (2):
1. One `grep` for the old wording and `isAncestorOrEqual` callers, plus `sed` of `destroy.go` ~1900-1915 (`seedPushCause`). My context held neither the other callers nor the post-edit text of that range.
