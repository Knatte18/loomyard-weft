# Card 1 — nested-modules

**Create:**
- `plan:internal/planparser#NestedModules` -> `func NestedModules(plan *Plan, through int, worktreeRoot string) []string`
- `plan:internal/planparser#ModuleOf` -> `func ModuleOf(modules []string, dir string) string`
- `internal/planparser/nestedmodules.go`
- `internal/planparser/nestedmodules_test.go`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** Add D6's one nested-module derivation, which the card gate (card 2), the new `verify-nested-module` check (card 3) and the caller scan (cards 4 and 5) all read.

`NestedModules` returns the sorted, slash-separated, worktree-relative directories below `worktreeRoot` that are nested modules for the plan as it stands once card `through` has run:
- every directory below the root holding a `go.mod` on disk, found by walking `worktreeRoot` and skipping directories whose name starts with `.` or `_` and directories named `vendor`, which the go tool never builds; `testdata` is walked, since a fixture module there is a real nested module a card can target;
- plus every directory whose `go.mod` a `Create` target of a card numbered at or below `through` names;
- minus every directory whose `go.mod` a `Delete` target of such a card names;
- with a `Rename` pair whose sides are `go.mod` files counted as a `Delete` of its Old side and a `Create` of its New side.
Map each ref to its file with `RefFile`, so a plain path and its self glyph read alike.
Walk cards in number order so a later card's Create or Delete wins over an earlier one.
The root's own `go.mod` (directory `.`) is never a nested module.
`through` of 0 counts no card, which is the on-disk set alone; the whole-plan form passes the plan's highest card number.
A `Move` card is not counted, D6's stated bound: its destination lives in `Intent` prose.

`ModuleOf` returns the innermost entry of `modules` that is `dir` or an ancestor of `dir` by whole path segments (`a/b` holds `a/b/c`, never `a/bc`), and `.` when none is, which is the root module.

Package doc for both goes on the functions; the file comment names D6's rule in one line.

`TestNestedModules` in the new test file is one table over a `t.TempDir()` tree, untagged and parallel: an on-disk nested `go.mod`; one under a `.`-prefixed, a `_`-prefixed and a `vendor` directory, all skipped; one under `testdata`, counted; a root `go.mod`, not counted; a `Create` of `x/go.mod` on a card at or below `through` counted and on a later card not; a `Delete` of an on-disk `go.mod` removing it; a `Rename` pair moving `a/go.mod` to `b/go.mod`; and `ModuleOf` rows for the innermost match, the root fallback and the sibling-prefix case.
The new top-level test raises `internal/planparser`'s row in `cmd/lyx/testdata/test-budget.yaml` by one.

**ImpactSummary:** new exported functions only; no existing caller changes, and the budget row moves by one.

**Commit:** 1: planparser derives nested modules for a plan through a card
