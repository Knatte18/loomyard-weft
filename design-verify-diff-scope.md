# Verify gates run only the touched trees and the repo-wide scans when a diff touches no production code

`verify-diff-scope` · Note · enhancement, loom, shed, config

## Seen

On 2026-10-18 the darn run `tokencount-segment-report` changed only `tools/tokencount`, yet its Darn-Write verify gate ran darn.yaml's `verify`, `lyx gate test ./... && lyx gate test --tags integration ./...`: both tiers over the whole module.
The same command runs for every darn and at loom's Merge-In and Publish, whatever the diff touches, so a change to a tool or to docs waits on the full suite, possibly up to three times.

## Decision (operator, 2026-10-18)

Scope by tree, never by import graph.
When every path the diff touches lies outside production code, such as `tools/` or `docs/`, the verify gate runs only the tests under the touched trees plus the repo-wide scans, the PATTERN scans and other tests that walk the whole tree.
When the diff touches anything else, the gate runs the whole-module command as today.

Why not the import graph: the compiler-visible dependents are exact, but tests also depend on files they read at run time (stencils, `contracts/`, fixtures), on tree-wide scans and on the built `lyx` binary, and none of that shows in the imports, so an import-scoped gate would let such breaks through.

## Open

- lyx must not hard-code Go or a tree layout, so the non-production trees, the scoped command and the scan packages are config of the module that owns the recipe; their names and shape are a design choice.
- Whether the final gate before landing stays whole-module even for such a diff; `dotnet-scoped-verify` asks the same question for a .NET repo.

## Operator view, not a decision (2026-10-20): darn's own gate is scoped

The operator expected darn's writer gate to run only what the change touched, as loom's card and batch gates do, and the whole suite only once at the end.
Today darn's one verify gate runs the hub's whole `verify` command (`darn.yaml`), at Darn-Write and at each Darn-Review fix round, then Publish runs it again when the tree changed, plus `publish_verify`; loom too runs its plan-level `## verify:` whole once per Webster attempt, but scoped gates run per card and per batch before it.
On OSL-1046 with Cortex XDR a whole run takes about 7.5 min, so a darn task pays it two or three times.
Idea: darn's writer and fix-round gates run the tests of the packages the diff touches and their reverse dependencies, and the whole suite runs once, at Publish.
