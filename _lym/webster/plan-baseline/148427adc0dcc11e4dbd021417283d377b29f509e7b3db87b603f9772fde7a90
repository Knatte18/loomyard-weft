# Card 15 — weavercli-reset

**Create:**
- `internal/weavercli/doc.go`
- `internal/weavercli/testmain_test.go`
- `plan:internal/weavercli#Command` -> `func Command() *cobra.Command`
- `plan:internal/weavercli#weaverCLI` -> `type weaverCLI struct`
- `plan:internal/weavercli#weaverCLI.resetCmd` -> `func (c *weaverCLI) resetCmd() *cobra.Command`
- `plan:internal/weavercli#resetVerb` -> `func resetVerb(out io.Writer, deps weaverengine.ResetDeps, stopStrands bool) int`
- `plan:internal/weavercli#outsideHubRefusal` -> `const outsideHubRefusal = "weaver: lym weaver reset runs only in a hub's task worktree, where Weaver runs as a loom row; way forward: run it from the task worktree of the run to reset"`
- `plan:internal/weavercli#TestResetVerb` -> `func TestResetVerb(t *testing.T)`

**Edit:**
- `cmd/lym#newRoot`

**Extends:**
- `//docs/overview.md`

**Uses:**
- `internal/webstercli#websterCLI.resetCmd`
- `internal/webstercli#TestResetCmd`

**Intent:** The `lym weaver reset` verb (D2; PATTERN-cli-cobra).

- `Command` returns the `weaver` cobra subtree, `Short` "the weaver implementer's verbs", whose one verb is `reset`.
  It resolves the hub location through `lymcwd` in its pre-run and refuses outside a hub with `outsideHubRefusal`, since Weaver runs only as a loom row; it builds no standalone wiring.
- `resetCmd` is `reset [--stop-strand]`, operator audience, `RunE` checking `clihelp.ShouldAbort` first, with a `Long` naming what the verb moves, restores, archives and commits, and the refusals' ways forward.
  It fills `weaverengine.ResetDeps` from `hubgeom.WeaverGeometry`, the shuttle runner and reed engine, the fabric handle's `ResetPairCode` and `CommitAnchoredPaths` over `weaverengine.DirRel()` and the plan directory, `fabricengine.AnchoredFileAt`, `fabricengine.AnchoredPathHasContent` for the pending-record read, and `fabricengine.AnchoredFileFirstParentHistory` over `weaverengine.StateRel()` mapped onto `weaverengine.StateVersion`.
- `resetVerb` runs `weaverengine.Reset` and prints one `internal/output` envelope: `ok` with the start commit, the restored plan commit and the archived names, or `err` with the refusal's text.
- `cmd/lym`'s `newRoot` mounts `weavercli.Command()` and its "Available modules" line names `weaver`.
- `doc.go` documents the package; `testmain_test.go` is untagged support running `tmuxkit.Main`.
- `docs/overview.md`'s package tree gains the `internal/weavercli/` line: `lym weaver reset`.

The flag lets a reset stop live strands its probes find; it is bounded to strands of this run's recorded attempts and fixers, found by their recorded output files, and nothing else.

`TestResetVerb` is tier0 over fake `ResetDeps` seams: the ok envelope's fields; a refusal's err envelope carrying its way forward; `--stop-strand` reaching `Reset` set; the pre-run outside a hub refusing with `outsideHubRefusal`.

**ImpactSummary:** Mounts one new module on the root command and names it in the root help; the cli-tree and registration scans then cover it.
