```yaml
lens: coverage
outcome: reported
```

Coverage lens: every decision and acceptance criterion, and every branch the cards add, checked against the card that carries it and the planned test row that reaches it.

Findings:

1. MEDIUM — D2, and the criterion that an empty own marker is refused by the push. Card 9's Intent has `retryWeftPush` refuse an empty own marker, but no planned row reaches that refusal. Card 9's `TestPushFailureClasses` rows cover only the marker-change replay stop and the fast-forward landing, and card 13 adds only a move-record row. Evidence: `_lyx/plan/09-weft-pull-and-push-stop-on-marker-change.md:31` and `:36`.

2. MEDIUM — the criterion that the pair teardown uses each worktree's own anchor in a hub with two anchors. Card 2 changes `pairteardown#Teardown.driverLockBusyError` and card 4 changes `pairteardown#Teardown.taskAnchor`, but neither plans a pairteardown test with a re-anchored pair. Card 3's `TestRemove_TearsDownNestedJunction` row exercises fabricengine's remove, not `pairteardown`. Evidence: `02-location-literal-scan.md:7,22` and `04-consumer-target-locations.md:5,22,26-28`.

3. MEDIUM — card 2's new branches have no row. Nothing reaches `pairLocation`'s new error return, `dobbyWatchDirs` skipping an unreadable slug with a warning, or `driverLockBusyError` falling back to the prime when `NamedWorktree` errs. The card's only test is the literal scan. Evidence: `02-location-literal-scan.md:21-22,26-28`.

4. MEDIUM — card 4's `ideengine#Menu` change (probing each worktree's `_lyx` at its own anchor) and the `fabriccli#runReconcile` `primeBaseDir` change (taken from the prime's own Location) are behavior changes with no planned row. `Teardown.taskAnchor`'s fallback to the bare path also has none. Evidence: `04-consumer-target-locations.md:9-10,22-24`; the Tests paragraph names only `TestDobbyGeometry` and `TestSpawnAnchoredScenario`.

5. MEDIUM — the criterion that re-anchoring one worktree keeps the exclude lines another worktree at the same anchor needs. Card 5's rows exercise unwire only. No card 15 or card 17 row checks the exclude lines after a move from `A`, with another worktree still wired at `A`. Evidence: `05-exclude-census-per-anchor.md:20-23`, `15-anchor-change-engine.md:54-60`, and the record's criterion "Unwiring or re-anchoring one worktree keeps the exclude lines …".

6. MEDIUM — the criterion that a child pair forked before its parent's change "still runs, and lands into the parent, whose marker stays unchanged". Card 17's row stands a records commit on the child's own weft branch in for the landing, so neither the landing nor the child's run is exercised. Evidence: `17-fabric-anchor-verb.md:60`.

7. MEDIUM — D7's Windows live-dobby refusal is unreachable in the planned tests. The refusal depends on running on Windows, but card 17's `anchorDeps` seam list has no OS seam, so the tier0 `TestChangeAnchor` on Linux can't reach that branch. Evidence: `17-fabric-anchor-verb.md:44,55`.

8. MEDIUM — D7 recovery: no row names the refusal when both `A/.lyx` and `B/.lyx` exist on a not-landed re-run (way forward: merge by hand). Card 15's rows list the first-run refusal table and the kill re-runs, but not this branch. Evidence: `15-anchor-change-engine.md:46` against `:54-60`.

9. MEDIUM — the D7 owner refusal has an allowed branch with no row. A worktree another machine owns, with `B` equal to the marker at the fetched upstream tip, must not be refused. Card 17's rows cover "each refusal above" only. Evidence: `17-fabric-anchor-verb.md:41,59`.

10. MEDIUM — the criterion that the verb spares "the pushes and board syncs outside the ended dobbies' process trees" has no row. `TestChangeAnchor` fakes the dobby end, and no row asserts the spared roles or a push outside the tree surviving. Evidence: `17-fabric-anchor-verb.md:49,59`.

11. MEDIUM — D5's markerless-value rule has untested branches. No row covers `treeAnchor`'s fallback, where no directory of the tree holds a wired name, or its byte-order tie at one depth. Card 8's rows cover only a pair whose content sits at `.` and a markerless prime. Evidence: `08-reconcile-migrates-markers.md:31,50`.

12. LOW — D2 against card 8's adopt path. An adopted weft branch whose committed own marker is empty falls into `treeAnchor`'s markerless branch, because `committedMarker` reports it as `""` found true and `treeAnchor` treats that as no marker. The path then commits a value over it instead of refusing, as D2 requires of every `fabricengine` path that acts on a marker. No row covers it either way. Evidence: `08-reconcile-migrates-markers.md:30-31,43`.

13. LOW — D3 says a consumer acting on a pair whose weft is gone uses no anchored path of that pair. Card 3 states it, but no row reaches the gone-weft branch. Evidence: `03-fabric-target-locations.md:44,52-55`.

14. LOW — D6 says a re-run whose move commit already landed goes on, but card 18 plans only the kill-before-commit re-run. Also, card 18's kill row has the hook fail inside `Topology.add`. That failure may roll back the whole pair, which would leave no present pair for `EnsureAnchor`. Evidence: `18-beam-create-at-an-anchor.md:33,36,44`.

15. LOW — D7: card 15's hook row covers a failure before the move commit, but not a failure inside an integration before the branch moves. The Intent claims both. Evidence: `15-anchor-change-engine.md:49,57`.

16. LOW — D7 step 8 starts a beam's dobby only when this machine owns it and its lock reads free. No row covers the not-owned or lock-held branch. Evidence: `17-fabric-anchor-verb.md:52,59`.

17. LOW — D7's log sink: the sink opening in the weft's lock directory while the record stands, and reopening at `B`, has no row. Evidence: `17-fabric-anchor-verb.md:50`.

18. LOW — D4: no row asserts the bootstrap guard's reworded message. Nothing reaches the probe's fetch error that is not a missing branch. Evidence: `06-clone-records-marker-on-prime-weft.md:44-45,52`.

19. LOW — D8: no row asserts that the prime's weft is migrated first, or that `lyx fabric unwire` leaves every marker. Evidence: `08-reconcile-migrates-markers.md:34,41,50`.

20. LOW — D10's checkout: the target weft branch that exists only as origin's remote-tracking ref has no row. `TestCheckout_RefusesAnotherAnchor` uses a local branch. Evidence: `10-checkout-and-drop-local-refuse-another-anchor.md:24,32`.

21. LOW — D7 dirt check: the exemption for the never-tracked `A/.lyx` on a first run has no row showing a `.lyx` file doesn't refuse. Only the re-run rows keep a `.lyx` file. Evidence: `15-anchor-change-engine.md:43,58`.

Tool calls: none; my context held the record, the cards and the exploration.
