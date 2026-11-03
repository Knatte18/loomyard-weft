```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked the change set for stale twins of the changed text. Findings:

1. **LOW** — `internal/pairteardown/doc.go:27` and the `endOwned` field comment at `internal/pairteardown/teardown.go:111`
   - Changed text: `endOwnedProcesses` now always spares `runproc.RoleTeardown`.
   - Summary: the package doc still says `runproc.End` "ends every process registered under the pair's slug … sparing the calling process and its ancestors". The `endOwned` field comment lists only the calling process, its ancestors and `spare`.
   - Evidence: both texts predate the change and omit the new fixed spare. A reader of the teardown doc would not learn that a running teardown child survives a later teardown, which is the behavior F5 introduced.

2. **LOW** — `docs/overview.md:740`
   - Changed text: the PATTERN and `remove.go` order was reworded to "tag locally first, delete local branches, then the child pushes the tag".
   - Summary: the batten teardown row still says teardown "first pushes an `archive/<slug>/<tip>` tag … then deletes that branch locally and on the remote".
   - Evidence: that is the old order the reviewer's F9 corrected in `PATTERN-fabric-git.md`. The twin in the overview was left alone.

3. **LOW** — `internal/fabriccli/anchor.go:411` and `anchordeps.go:447`
   - Changed text: new `RoleTeardown` role, registered under the slug owner.
   - Summary: the anchor move's spare list (`RolePush`, `RoleSync`) and the `dobbyPassInFlight` spared-pid set were not extended to the teardown role.
   - Evidence: both spare the detached children that hold the pair's work. The test `anchor_test.go:506` pins the exact list `push,sync`. This is uncertain: the move concerns a live worktree, while the teardown child exists only after the pair is removed. I checked nothing further on whether a move can reach a slug that has a running teardown child.

4. **LOW** — `internal/fabricengine/doc.go` (the paragraph near line 603) and `remoteteardown.go` (the `RunTeardownChild` doc)
   - Changed text: a failed step now yields one line with a clause per failed step, and an unreadable job is mailed too.
   - Summary: the paragraph says "one line naming the slug, the operation and the way forward". It does not say that every failed step is named, or that a job the child cannot read is reported.
   - Evidence: the text is still true but incomplete after F1 and F7. This is minor.

Consistent after the change:
- The `ErrLeaseRejected` handling matches the doc line "a refusal or lost lease is logged as kept and is no failure".
- The `lym fabric` help and doc paragraphs match the child's new pause exemption.
- `teardownLockRefusal` still correctly points the operator to `lym tender status` to end a stuck teardown child.
- `PATTERN-push-both-sides.md:9` already describes the weft-first order.

Tool calls:
1. `grep` for `RolePush` and for old archive-order wording. The context did not show other users of the role or twins of the old wording.
2. `sed` of `internal/fabricengine/doc.go` lines 595-612, plus a grep of tendercli role handling. The context did not hold the engine doc paragraph. The tender grep found no role enumeration.
3. `grep` for process-end sparing docs in pairteardown, tendercli and docs. The context did not show how those docs describe the end.
