```yaml
lens: fix-consistency
outcome: reported
```

Checked with the fix-consistency lens: whether each changed text still agrees with the sibling cards, decisions, rows and cross-references it touches.

1. LOW — `_lym/plan/06-kwinplace-scripts-readers.md:89` is a stale twin of the overview's "contract-test-tier" decision, which now reads "The contract tests against a real KWin are `tier3`" (plural). Card 8 now splits the contract work into `TestContract_TheRealKWinAnswersAsRecorded` and `TestContract_TheRealKWinAppliesAsRecorded`, but card 6 still says "The contract test that drives a real KWin is card 8's", in the singular. `01-kwin-probes.md:28` ("card 8's contract test compares them with a live KWin") is still correct, since the `neve-qdbus` comparisons are all in `AnswersAsRecorded`.

2. LOW — card 12's `RefusedTaskMode` row now reads "on a task worktree of its own, with the variable set empty, so no other row resolves its mode". The inserted clause sits between the worktree and its "so" clause. The sentence now seems to say that the empty variable is what stops other rows resolving its mode, when the separate worktree is the reason. Moving "with the variable set empty" after the "so" clause fixes it.

Agreement verified, no finding:
- In card 11, step 5's off path and the "Under mode `off` … snapshot alone" line agree with the new landed-desktop and output bullets. The new `tilingOutput` parameter is fed by step 2's read.
- Card 9's `ReportUpdate` parenthesis, the first `updatePlaces` rule and the new `TestUpdatePlaces` row agree with card 11's per-path rule. The fix leaves `Settle`'s rule as it was, and card 9's "record-only run" wording matches it.
- The overview's run-registration decision matches card 12's `Spawn` text and its rows. Cards 4, 13 and 14 make no claim about registration that the fix leaves stale.

Tool calls:
- One grep over cards 01–07, 10, 13, 14 and 19 for SIGTERM, RolePlacement, teardown, landed, "nothing moved" and "contract test", to find twins of the changed text in cards my context did not hold.
