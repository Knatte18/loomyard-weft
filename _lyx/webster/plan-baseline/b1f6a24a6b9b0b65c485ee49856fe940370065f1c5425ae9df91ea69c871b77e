# Card 1 — impactset-cmd-lyx-join

**Edit:**
- `internal/impactset#moduleGraph.impactedSet`
- `internal/impactset#moduleGraph.addBinaryBuildingTests` -> `func (g moduleGraph) addBinaryBuildingTests(set map[string]bool) []string`
- `internal/impactset#targetsFor`
- `internal/impactset#Targets`
- `internal/impactset#TestTargetsFor`
- `internal/impactset#testGraph`
- `internal/impactset/doc.go`

**Intent:** D5: the impacted set adds every package whose tests import `internal/testkit/lyxbin` only when the diff changes the `cmd/lyx` main package itself, never for a change to one of its dependencies.

`moduleGraph.impactedSet` replaces its `cmd/lyx` closure walk with one check: when `changed` holds the import path of the package at `commandDir`, it calls `addBinaryBuildingTests`.
Every other package reaches the set through `reaches` alone, so a binary-driving test package joins only when its own build or test closure reaches a changed package.

`addBinaryBuildingTests` also returns the sorted module-relative directories of every package whose test imports name the lyxbin package, so the same scan feeds the new field below; `impactedSet` ignores the return.
`Targets` gains the field `BinaryTests []string`: the sorted directories, relative to the worktree, of every package whose tests import `internal/testkit/lyxbin`, inside the set or not, empty with Whole.
`targetsFor` fills it on every non-Whole answer, a Markdown-only diff included, from a scan over the graph that does not touch the set (call `addBinaryBuildingTests` on a throwaway map).
Card 3's classifier reads it to say a failing package drives the `lyx` binary (D2); `internal/gatecli` reads only `Whole`, `Fallback`, `Base`, `Modules` and `Guards`, so it is unaffected.

`TestTargetsFor`:
- the row "one changed package yields it and its reverse dependencies" over `internal/a/a.go` no longer holds `./internal/e`, which reaches `a` only through the binary;
- the row "a change to a dependency of cmd/lyx pulls in every package that builds the binary" is reworded to "a change to a dependency of cmd/lyx leaves out a package that reaches it only through the binary" and wants `./cmd/lyx`, `./internal/b`, `./internal/c` without `./internal/e`; it is the graph form of a diff confined to `internal/planglyph` leaving `internal/reedcli` out;
- a new row, "a change to the cmd/lyx main package pulls in every package that builds the binary", changes `cmd/lyx/main.go` and wants `./cmd/lyx` and `./internal/e`;
- every non-Whole row's `want` gains `BinaryTests: []string{"internal/e"}`, and the Whole rows keep it nil, so the field's fill and its absence on a fallback are both pinned;
- the table gains an optional `graph` field, `testGraph()` when unset, and a new row over a graph without the `internal/testkit/lyxbin` package changes `cmd/lyx/main.go` and wants the set without `./internal/e` and an empty `BinaryTests`, reaching the scan's early return.

`testGraph`'s doc comment says `e` builds the binary in its tests and reaches nothing else, so it joins only for a change to `cmd/lyx` itself.
`doc.go` replaces the sentence "When a changed package is a dependency of `cmd/lyx`, every package whose tests import `internal/testkit/lyxbin`, which builds the binary, joins the set." with: when the diff changes the `cmd/lyx` main package, which no test imports and so the graph cannot see, every such package joins; any other change reaches them through the import graph only.
It adds that `Targets.BinaryTests` lists those packages, and extends the `Bound:` paragraph: a binary-driving test that reaches a change only through a subcommand its own package does not import leaves the narrowed run, and the whole runs at Merge-In, Publish and Finalize still run it.
The `Targets{Whole, Fallback, Base, Modules, Guards}` enumeration in `doc.go` gains `BinaryTests`.

The change applies to every consumer of the set: darn's gates, loom's round gate and every computed-scope `lyx gate test`.
It skips binary-driving packages for a diff that reaches them only through the binary, bounded by the whole-tree runs at the landing sites, which `doc.go`'s bound states.

**ImpactSummary:** pure-core change inside `internal/impactset`; the set narrows for every computed-scope consumer, and the one new `Targets` field is additive, read only by card 3.
