# Card 6 — internal/impactset: the exported diff reads

**Create:**
- `plan:internal/impactset#ChangedPaths` -> `func ChangedPaths(ctx context.Context, worktree, base string) ([]string, error)`
- `plan:internal/impactset#AncestorBase` -> `func AncestorBase(worktree, base string) (string, bool)`

**Extends:**
- `internal/impactset/doc.go`

**Intent:**
`impactset` exports its two diff reads for the `dotnet` scoper and, in card 11, the inert check: `ChangedPaths` is `changedPathsInTree` (tracked changes from the base to the working tree, a rename at both paths, and untracked files through `git ls-files --others --exclude-standard`), and `AncestorBase` is `resolveAncestorBase`.
`impactset`'s doc names the two exports.

Tests:
The card adds no test, since each export only renames an unchanged, already tested function; card 7's `TestDeriveDiff` rows for an untracked `.cs` file, a rename and a non-ancestor base, and card 11's `inert_paths` rows of `TestGateTest_Scopes`, exercise them through their callers.

**ImpactSummary:** two new exported wrappers in `impactset` over unchanged functions; no existing caller changes.
