# Card 2 — fabric-checkout-states

**Create:**
- `plan:internal/fabricengine#CheckoutState` -> `type CheckoutState struct`
- `plan:internal/fabricengine#CheckoutStates` -> `func CheckoutStates(l *lymcwd.Location) (code, records CheckoutState, err error)`
- `plan:internal/fabricengine_test#TestCheckoutStates` -> `func TestCheckoutStates(t *testing.T)`

**Extends:**
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/fabricengine#RecordsWorktree`
- `internal/fabricengine#trackedDirtyPaths`
- `internal/gitrepo#Repo.UntrackedFiles`

**Intent:** Add the read-only checkout reading the judge's snapshot (D9) needs, as a general seam that names no drift concept.

`CheckoutState` holds `Root` (the checkout's absolute directory), `Head` (its HEAD commit), `Changed` (the tracked paths that differ from HEAD, staged, unstaged or deleted, as `trackedDirtyPaths` reads them) and `Untracked` (every untracked path outside the ignore rules and `.git/info/exclude`, as `Repo.UntrackedFiles` reads them), all slash-separated and relative to `Root`, sorted, never nil.
`CheckoutStates` returns that state for the warp (code) checkout and for the weft (records) worktree `RecordsWorktree` names.

`TestCheckoutStates` (tier1, hubforge `CopyHub`, `t.Parallel`) covers: a clean pair reads both heads and empty lists; a modified tracked code file is listed under the code side's `Changed`; a new file in the weft worktree is listed under the records side's `Untracked`; a file the ignore rules cover is in neither list.
The doc.go paragraph listing fabric's git reads gains `CheckoutStates`.

**ImpactSummary:** Adds one read-only fabric function and a type; nothing existing changes.
