# Card 2 — fabric-checkout-states

**Create:**
- `internal/fabricengine#CheckoutState`
- `internal/fabricengine#CheckoutStates`
- `internal/fabricengine#WeftPathsChangedSince`
- `internal/fabricengine_test#TestCheckoutStates`

**Extends:**
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/fabricengine#RecordsWorktree`
- `internal/fabricengine#trackedDirtyPaths`
- `internal/gitrepo#Repo.UntrackedFiles`
- `internal/gitrepo#Repo.ChangedFilesSince`

**Intent:** Add the read-only checkout readings the judge's snapshot (D9) needs, as general seams that name no drift concept.

`CheckoutState` holds `Root` (the checkout's absolute directory), `Head` (its HEAD commit), `Changed` (the tracked paths that differ from HEAD, staged, unstaged or deleted, as `trackedDirtyPaths` reads them) and `Untracked` (every untracked path outside the ignore rules and `.git/info/exclude`, as `Repo.UntrackedFiles` reads them), all slash-separated and relative to `Root`, sorted, never nil.
`CheckoutStates` returns that state for the warp (code) checkout and for the weft (records) worktree `RecordsWorktree` names.
`WeftPathsChangedSince` returns the weft paths whose committed content differs between `rev` and the weft worktree's HEAD, read through `Repo.ChangedFilesSince` on that worktree, slash-separated and relative to its root as `CheckoutState`'s lists are, sorted, never nil.

`TestCheckoutStates` (tier1, hubforge `CopyHub`, `t.Parallel`) covers: a clean pair reads both heads and empty lists; a modified tracked code file is listed under the code side's `Changed`; a new file in the weft worktree is listed under the records side's `Untracked`; a file the ignore rules cover is in neither list; after a weft commit changing one file, `WeftPathsChangedSince` of the earlier head names that file alone, and of the current head names none.
The doc.go paragraph listing fabric's git reads gains `CheckoutStates` and `WeftPathsChangedSince`.

**ImpactSummary:** Adds two read-only fabric functions and a type; nothing existing changes.
