# Card 3 — fabric-bolt-dir-sweep

**Edit:**
- `internal/fabricengine/anchormarker.go`
- `internal/fabricengine/boardweft.go`
- `internal/fabricengine/bolt.go`
- `internal/fabricengine/cleanup.go`
- `internal/fabricengine/clone.go`
- `internal/fabricengine/codebinding.go`
- `internal/fabricengine/destroy.go`
- `internal/fabricengine/doc.go`
- `internal/fabricengine/fabric.go`
- `internal/fabricengine/hubpull.go`
- `internal/fabricengine/hubworkspace.go`
- `internal/fabricengine/originreconcile.go`
- `internal/fabricengine/pendingflags.go`
- `internal/fabricengine/pull.go`
- `internal/fabricengine/pushclassified.go`
- `internal/fabricengine/reconcile.go`
- `internal/fabricengine/recordsgit.go`
- `internal/fabricengine/remove.go`
- `internal/fabricengine/shortnamecommit.go`
- `internal/fabricengine/slug.go`
- `internal/fabricengine/spawn.go`
- `internal/fabricengine/stencilcommit.go`
- `internal/fabricengine/stencilhistory.go`
- `internal/fabricengine/teardownlock.go`

**Uses:**
- `plan:internal/fabricengine#BoltDir`
- `plan:internal/fabricengine#BoltDirName`

**Intent:** Move fabric's own uses of the board names that mean the Bolt checkout onto the Bolt names (D9), with no behaviour change.

- Every `BoardDir(...)` call in these files means the Bolt checkout (the weft:main worktree, its locks, its anchor marker, its binding and shortname records, its stencils, its push work directory, the parent-lock store under it), so each becomes `BoltDir(...)`; every `BoardDirName` use (the hub-structure checks in `fabric.go`, `clone.go`, `destroy.go`, `cleanup.go`, and `pull.go`'s `weftEndedByOrigin`, which joins it by hand) becomes `BoltDirName`, and `weftEndedByOrigin`'s hand join becomes `BoltDir(l.HubPath)`.
  Fabric keeps no use of `BoardDir` or `BoardDirName` afterwards; the board's own files are read through the replay seam (card 12).
- Every message these files build that names the directory takes it from `BoltDirName` through a format verb, as `codebinding.go`'s "edit %s in the hub's %s worktree and commit" and `destroy.go`'s "no %s entry and no weft sibling" do; a message that spelled `_board` as text, such as `codebinding.go`'s re-point refusal, gains the verb.
- Every comment that spells the path (`<Hub>/_board`, `_board/.lym`, `_board`'s branch, its records sibling) names "the Bolt checkout" (or "the Bolt checkout's `.lym`", "the Bolt checkout's branch") instead; `doc.go`'s sections name `BoltDir(hub)` and "the Bolt checkout".
- A grep for the whole token `_board` over the package's non-test files then finds nothing; card 7's guard enforces it.
- The existing fabric suites over clone, cleanup, destroy, push, pull and reconcile cover the unchanged behaviour; a test that pins a message with the literal moves to the constant in card 4.

**ImpactSummary:** 24 fabricengine files, comment and constant swaps only; every path and message value is unchanged, so no caller or test outcome moves.
