```yaml
lens: fix-proposal
outcome: reported
```

Proposals checked: F1, F2, F3, F4, F6, F7, F8 and F9. The fixer applied F5 with its own change, so it is outside this lens.

**MEDIUM: F3 drops the only test of a lease extension**
- **Changed text:** card 10's **Create:** list and Tests list, which drop `TestDetachStore`.
- **Summary:** The dropped test pinned that `writeLease` keeps the later of two deadlines whichever comes first. The surface rows the review cites pin only one order: an earlier deadline written over a later one, in the row where a 1s-cap call joins a 6s-cap call. No remaining row pins a joiner moving the lease later.
- **Concrete case:** suppose `writeLease` writes only when no lease exists yet. It passes the 1s-joins-6s row. It also passes every other `TestGateTest_DetachedRun` row, because they run under the default 120s grace, so no lease ever runs out before a join would need to extend it. The 1s-grace row makes a single call. A runner would still be abandoned at the first call's lease even though a later call renewed it.
- **Possible fix:** give card 11's `TestGateTest_DetachedRun` a row with a 1s cap and a 1s grace. A second call joins before the first lease passes, and the runner outlives the first call's deadline plus grace.

**LOW: F4's new reason names an ambiguous "card-verify runner's timeout"**
- **Changed text:** card 14's Intent, the new `verify-gate-no-wait` reason and the Detail text.
- **Summary:** The proposed reason holds against the code. Per `internal/cardverify/doc.go:10-11`, a line that is not a `--wait` gate call takes one slot, runs under the Runner's per-line timeout, and passes the slot on to a nested gate call through `HolderEnv`.
- **Why it misleads:** after card 13, "the card-verify runner" also names the detached runner (`runDetached`). That runner is bounded by its lease, not by a timeout. Read that way, the spec text and the finding's Detail point at the wrong bound.
- **Possible fix:** say "under `cardverify`'s per-line timeout".

Tool call: read `internal/cardverify/runner.go` `IsGateCall` and `internal/cardverify/doc.go` to test F4's new reason against the code. My context did not hold cardverify's slot rule for a gate call without `--wait`.
