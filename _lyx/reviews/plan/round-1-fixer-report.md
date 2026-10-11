# Plan round 1 — fixer report

Every finding's premise was checked against the code and holds; each is fixed as proposed.

## Changes

- **F1** (`10-review-refusals-name-status.md`): added `plan:internal/loomcli#TestParentReviewExchange_RewriteReviewedThenCapRejectHalts` to the Edit list.
  The Tests paragraph now says that test drives `reviewState.preRun`'s choice through the real CLI before arrival 1: `review notify <slug>` from the prime names `lyx batten status <slug>` and `from the prime`, and `review notify` with no slug from the task worktree names `lyx loom status` and no `lyx batten status`.
- **F2** (`12-gate-recheck-throttle.md`): dropped the `TestGatePending_IdleRecheckThrottled` Create bullet, the `test-budget.yaml` Edit bullet and the budget-raise sentence.
  Added `TestGatePending_RejectAfterPendingRepromptsOnce` to the Edit list; the Tests paragraph says it asserts no gate call on the idle tick one poll after the pending evaluation and a call on the tick its `STOP:turn2` arrival lands, two fake-clock seconds later, inside the throttle window.
  `00-overview.md`'s test-budget decision no longer lists card 12 among the cards raising a row.
- **F3** (`05-parent-review-no-pass-at-cap.md`, `06-parent-review-last-gate.md`): card 5 drops the `TestResolveGateSpec_ParentReviewElementRefusals` Create bullet and edits `TestResolveGateSpec_Rejections`, which builds over `parentReviewEnv(t)` and gains the `pass_on_cap: true` and `pass_on_cap: false` rows, each asserting `way forward:`.
  Card 6 edits `TestResolveGateSpec_Rejections` (the `parent-review` then `discussion` row) and `TestResolveGateSpec_ParentReviewResolvesMustPassMayHold` (config `discussion` then `parent-review`, two entries with `parent-review` last), and no longer extends card 5's table.
  `00-overview.md`'s test-budget decision now says card 5 deletes one shedrecipe test rather than swapping it.
- **F4** (`01-engine-prompt-cache-lifetime.md`): `TestPromptCacheLifetime`'s table covers only the parse and the passed-through error: the 5m default, a role mapped to `1h` and an invalid default; the unmapped-role and empty-role rows are gone.
- **F5** (`01-engine-prompt-cache-lifetime.md`): the test now lives `in prepare_test.go beside the TTL rows and the ttlConfig helper`.

## Disputed

None.

## Deferred

None.
