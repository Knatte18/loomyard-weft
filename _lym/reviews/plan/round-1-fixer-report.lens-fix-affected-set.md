```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the fix covers every member of each set-based finding (callers, tests, sites).

Unchecked findings: none. I reran the search for every set-based finding (F1, F6, F7, F8, F9, F10, F11, F13). F11's caller set is the parent's earlier grep, which my F10 search confirms.

no findings

- **F7:** The callers of `PushWithRetry` are `push.go:139` (`pushPlainWithRetry`), `archive.go:131`, `pushretry_test.go:98` and `pushretry_integration_test.go:148`. The callers of `retryablePushRefusal` are `pushretry.go:101` and `pushretry_test.go:51`. Card 6 moves the archive push to `PushWithRetryReasons`, and card 1 keeps `PushWithRetry`'s signature for the rest. The re-signed predicate's only test caller, `TestRetryablePushRefusal_…`, is on card 1's Edit list.
- **F9:** The hand-restore text appears in exactly three places: the doc at `wire.go:203`, the refusal at `wire.go:224` and the assertion at `wire_test.go:707`. Card 10's Edit list and test bullet now cover all three.
- **F8:** `removeFields` has two callers, `fabric.go:1151` (`runRemoveWithFlag`, now under Edit) and `removefields_test.go:127` (`TestRemoveFields`, under Edit).
- **F6:** The only other battencli text naming `lym fabric remove` is `wire.go:317,321`, the Worktree-Create `incompletePairRemedy`. The `d3-sites-outside-fabric` decision already exempts it.
- **F10:** The production callers of `PushPairAnchored` are `wiring.go:214`, `landingdeps.go:329` and `weft_verbs.go:295`. Card 5 covers all three: `commitRecordsVerb`, the landing's `PushTask` and `addWeftVerbs`. Outside fabricengine, the only consumer of `ErrPairPushRunning` is battencli `teardownRefusal` at `wire.go:408`. It handles only `Remove`'s error, so the new skip error never reaches it.
- **F13:** The other doc texts on the teardown wait and the archive tag are each either rewritten by a card or not made false:
  - Rewritten by a card: `teardownlock.go:1,23,40` (card 5), and `archive.go:1,21,47` and `topology.go:26` (card 6).
  - Still true: `remove.go:207` ("detached pushes hold this lock shared"), `archive.go:96`, and `doc.go:598,758`.

Calls made:
1. Grepped the `PushWithRetry` and `retryablePushRefusal` call sites (F7). My context held only `archive.go`'s call and `push.go`, not the test or integration callers.
2. Grepped for the hand-restore refusal wording (F9), to find every place that pins it beyond the two the review named.
3. Grepped the `removeFields` callers and battencli's `lym fabric remove` texts (F8, F6), to complete both sets.
4. Grepped fabricengine and pattern docs for the teardown-wait and archive-tag wording (F13), to find sibling docs the review did not list.
5. Grepped the consumers of `ErrPairPushRunning` (F10). My context did not show who reads the sentinel the skip now returns.
6. Read `wire.go:395-415` (F10), to see what battencli's `ErrPairPushRunning` branch wraps.
