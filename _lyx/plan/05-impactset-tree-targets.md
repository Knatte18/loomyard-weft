# Card 5 — impactset-tree-targets

**Create:**
- `plan:internal/impactset#Targets` -> `type Targets struct`
- `plan:internal/impactset#ModuleTargets` -> `type ModuleTargets struct`
- `plan:internal/impactset#DeriveTargets` -> `func DeriveTargets(worktree, base string) (Targets, error)`
- `plan:internal/impactset#changedPathsInTree` -> `func changedPathsInTree(worktree, base string) ([]string, error)`
- `plan:internal/impactset#targetsFor` -> `func targetsFor(graph moduleGraph, changedPaths []string, guards map[string][]string) Targets`
- `plan:internal/impactset#TestTargetsFor` -> `func TestTargetsFor(t *testing.T)`

**Edit:**
- `internal/impactset#TestDerive_OverARepository`
- `internal/impactset/doc.go`

**Uses:**
- `internal/impactset#testGraph`
- `internal/impactset#writeRepoFile`
- `internal/impactset#commitAll`
- `internal/impactset#packageOfPath`
- `internal/impactset#packageArgs`

**Intent:**
Make `impactset` the `go` scoper (D3): a target set over the working tree, not a command.

`DeriveTargets(worktree, base)` returns `Targets{Whole bool, Fallback string, Base string, Modules []ModuleTargets, Guards map[string][]string}`, with `ModuleTargets{Dir, Packages []string}`, the module directory worktree-relative (`.` for the root module) and its packages module-relative package arguments in the order the graph lists them.
It diffs `base` to the working tree, not to HEAD: `changedPathsInTree` reads `git diff --name-status -z --find-renames <base>` through `gitexec.Run` for tracked changes, staged or not, a rename counted at both paths, plus `git ls-files --others --exclude-standard -z` for untracked files that are not ignored.
It falls back to `Whole` with the reason wherever `Derive` falls back today: no `go.mod` at the worktree root, an empty or non-ancestor base, a change to `go.mod` or `go.sum`, a changed path that is neither `.md` nor under a Go package directory (a nested module's file included, since the root `go list` does not list it), and a failed `go list`.
The command-length fallback does not apply, since no command is spelled.
Otherwise `targetsFor`, the pure core, takes the impacted set as `commandFor` computes it, the binary-building tests included, into the root module's packages, every impacted package kept, test-less ones included.
A diff touching only `.md` files yields no module and no package, which the gate reads as an empty set (card 9).
`Guards` holds the module's `//lyx:guard` tests in packages outside the set, directory to sorted names, so the gate's guard step (D5) runs them by name; a guard scan error is returned as the `*GuardScanError` it is today.
`Derive` stays until its last caller moves (card 10).

`doc.go` documents `DeriveTargets` beside `Derive`.

Tests: the new `TestTargetsFor` covers the pure core over `testGraph` and carries every `TestCommandFor` row but the command-length one, recast as target sets, since card 10 deletes `TestCommandFor` with `commandFor`: a changed package and its importers with tmux and llm test imports, the binary-building packages pulled in by a `cmd/lyx` dependency and left out otherwise, a rename putting both packages in the set, an `.md`-only change yielding no module, a guard name held by two outside packages listed once, a non-Go file mapped to its enclosing package, the not-`.md` and deleted-package fallbacks, the `go.mod` and `go.sum` fallbacks, and a test-less impacted package kept.
`TestDerive_OverARepository` gains rows calling `DeriveTargets` over a scratch repository: an uncommitted edit and an untracked new file each select their package, an ignored file selects nothing, an `.md`-only working-tree change yields the empty set, a file of a nested module falls back to whole, a `go.sum` edit falls back to whole, and a misplaced `//lyx:guard` marker returns the `*GuardScanError`.

**ImpactSummary:** additive exports beside `Derive`; no caller changes in this card.
