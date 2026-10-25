# Card 9 — Weft pulls and replaying pushes stop on a marker change

**Edit:**
- `internal/fabricengine#Fabric.Pull`
- `internal/fabricengine#Fabric.pullWeftArm`
- `internal/fabricengine#retryWeftPush`
- `internal/fabricengine#Fabric.fastForwardWeft`
- `internal/fabricengine_test#TestFastForwardPair_MovesBehindPairToUpstream`
- `internal/fabricengine_test#TestPull_WeftDivergedWarpAdvancesCleanly`
- `internal/fabricengine_test#newDivergedWeftFixture`
- `internal/fabricengine_test#TestPushFailureClasses`
- `internal/fabricengine#pushJob.classifyFailure`
- `internal/fabricengine#pushJob.record`
- `internal/fabricengine#FailedPush`
- `internal/fabricengine#FailedPushWayForward`

**Create:**
- `plan:internal/fabricengine#ErrAnchorChange` -> `var ErrAnchorChange error`
- `plan:internal/fabricengine#AnchorStopError` -> `type AnchorStopError struct`
- `plan:internal/fabricengine#PushFailureAnchorChange` -> `const PushFailureAnchorChange PushFailureClass = "anchor_change"`
- `plan:internal/fabricengine#weftAnchorStop` -> `func weftAnchorStop(weftPath, head, upstream string, names []string, fallback string, reset bool) (wantAnchor string, stop bool, err error)`

**Uses:**
- `plan:internal/fabricengine#treeAnchor`
- `internal/fabricengine#refuseEmptyOwnMarker`
- `internal/fabricengine#WeftStopError`
- `internal/fabricengine#Fabric.settleUnmovedWeft`

**Intent:**
D10's one marker comparison and the pull and push that call it.
`weftAnchorStop` is the one `fabricengine` function holding the comparison; the origin reconcile (card 11) and the drop-local reset (card 10) call it too.
It reads `treeAnchor` of `head`, of `upstream` and of their merge base, each with `fallback`.
It stops when `upstream` has commits `head` lacks and the two tips' anchors differ: the fast-forward target, and the replay result built on `upstream`, would then leave a marker other than the current one checked out, or carry local commits across a marker change.
A `head` ahead of an `upstream` it fully contains never stops, so a fast-forward push of a re-anchor not yet pushed goes through; a markerless tip compares by D5's value, so markerless commits whose content sits at the local marker compare equal.
`wantAnchor` is the upstream tip's anchor when the local tip's anchor equals the merge base's, else the local tip's, for the way forward `lyx fabric anchor <slug> --set <wantAnchor>`, which integrates.
With `reset`, for a caller that replaces the checked-out tree with `upstream`'s (card 10's drop-local), it ignores ancestry: it stops whenever the two tips' anchors differ, a `head` ahead of `upstream` included, and `wantAnchor` is the upstream tip's anchor.
The pull, the push and the origin reconcile pass `reset` false.

`Fabric.pullWeftArm`, once the weft has an upstream, first refuses an empty own marker (`refuseEmptyOwnMarker`) as a weft stop, then fetches the weft branch and runs `weftAnchorStop` on `HEAD` and the fetched upstream before `PullWeft`; a stop returns a `*WeftStopError` whose `Err` wraps `ErrAnchorChange` and whose `WayForward` names the `--set` from the pair's code worktree.
`Fabric.Pull` returns such an anchor stop, and an empty-marker stop, before the warp arm runs, so both sides stay unchanged; other weft stops keep today's order.
`Fabric.fastForwardWeft`, the weft pull `FastForwardPair` runs ahead of its code side (a landing's Finalize reaches it), makes the same checks once the weft has an upstream: it refuses an empty own marker, then fetches the weft branch and runs `weftAnchorStop` on `HEAD` and the fetched upstream before `PullWeft`.
A stop skips the weft pull with its existing warning, which then names the way forward, and the code side goes on as it does after any other weft pull failure.
`retryWeftPush` refuses an empty own marker, and after its fetch, when the upstream has commits the weft lacks, runs `weftAnchorStop` before the replay; a stop returns an `*AnchorStopError` and changes nothing.
`AnchorStopError` carries `WayForward string` (the `--set`, or for an empty marker the file's restore) and reports `errors.Is(err, ErrAnchorChange)`.
`pushJob.classifyFailure` gains a case ahead of the `gitrepo.ErrPushRejected` one: an error that `errors.As` an `*AnchorStopError` is recorded as `PushFailureAnchorChange` with the stop's way forward, so the record never carries the not-fast-forward class, whose way forward names a `git merge` that would carry the other marker in by hand; `pushJob.record` copies the way forward into `FailedPush`'s new `WayForward string` field (`json:"way_forward,omitempty"`, a tolerant record), and `FailedPushWayForward` returns that field for the new class.
The fallback each passes is the pair's `lyxcwd.NamedWorktree(...).AnchorRel`.

Tests:
`TestPull_WeftDivergedWarpAdvancesCleanly` gains rows (with `newDivergedWeftFixture` taking an optional marker per side, and an empty `localRel` meaning no local weft commit): origin's commit changes the marker while the weft has no local commit, and both sides change it differently, each return a `*WeftStopError` wrapping `ErrAnchorChange` naming `lyx fabric anchor` and the right anchor, with the weft `HEAD` and the warp `HEAD` unchanged; origin gains markerless commits whose content sits at the local marker and the pull replays with no stop; the weft is ahead of origin with a re-anchor not yet pushed and the pull is no stop; an empty own marker stops naming `.lyx-anchor`.
`TestPushFailureClasses` gains rows: a push that must replay onto an upstream carrying another marker fails with `ErrAnchorChange`, the weft `HEAD` and origin's tip unchanged, and leaves one failed-push record of class `anchor_change` whose `FailedPushWayForward` names `lyx fabric anchor` and no `git merge`; a push rejected while the weft's own marker is empty fails the same way, its record's way forward naming `.lyx-anchor`.
`TestFastForwardPair_MovesBehindPairToUpstream` gains a row: an upstream weft carrying another marker leaves the weft `HEAD` unchanged while the code side still lands on its upstream tip.

**ImpactSummary:** the weft arm of `Fabric.Pull`, the fast-forward's weft pull and the push retry gain one pre-move check each, and the failed-push record gains one class and one optional field read only for that class; a hub whose wefts carry one marker pulls, pushes and records failures exactly as today.
