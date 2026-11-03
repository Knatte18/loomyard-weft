```yaml
lens: code-affected-set
outcome: reported
```

**Task:** per change the diff makes, the set it forces outside its own lines, and whether each member is covered.

**Set per change**

1. `fabricengine.Clean(l, weftExempt ...string)`: callers are `preflight.CheckResolved` (passes the exempt list) and the `fabricengine` and `loomcli` tests. The tests pass nothing and keep the unscoped check, so they are covered. `preflightshed` reaches it only through `preflight.Check`, so it inherits the exemption. The only doc line that names `Clean` is `fabricengine/doc.go:1022`, a read-only-callers list that stays true. Covered.
2. `worktreeDirty(..., pathspec ...string)`: every caller passes no pathspec and behaves as before. The `export_test.go` re-exports are unaffected. The `--` goes after `--untracked-files=no`, which is correct. Covered.
3. `dirtyReason(label, dir, pathspec...)`: the only non-test caller is `Clean`. Covered. Its doc comment is unchanged in meaning.
4. Preflight now imports `shedrun`. No `preflight` or `shedrun` entry in `PATTERN-leaf-packages` or `PATTERN-fabric-isolation` is affected, and `go list` shows `shedrun` does not import `preflight`, so there is no cycle. Fabric learns nothing of shed, since the paths arrive as an argument. Covered.
5. Docs: the new paragraph in `internal/preflight/doc.go` matches the code. The `Clean` doc comment mentions only the weft-side scope. The `doc.go` header "deliberately read-only" stays true. Covered.
6. PATTERN declared changes: the rubric's declared-changes list is empty and the diff changes no `PATTERN.md` or `pattern/` file. Nothing to land. Covered.
7. Commit messages: no `Pattern-Departure:` trailer is present and none is needed.

**Findings**

- **LOW, consistency**: `internal/fabricengine/warpclean.go:20` (the `Clean` doc comment).
  - The new `weftExempt` sentence is placed between the "weft-side check is skipped when…" line and the `Returns` line, and it does not say the exemption applies only to the weft side.
  - The doc says "anchor-relative weft paths", which is adequate, so this is a doc-polish nit and not a defect.
  - Evidence: the diff hunk at `warpclean.go:17-22`.
- **LOW, consistency**: `internal/shedrun/paths.go:345` (the `RunsRootRel` doc comment).
  - The comment says the function "exists so fabricengine's Add can drop the whole run-records tree".
  - It now also serves `preflight`. The comment is stale by omission, though not false.
  - Evidence: `shedrun/paths.go:343-349` and the new use at `preflight.go:22`.
  - The `SettleRecords` doc at `paths.go:337-338` also says "a dirty-path check counts it as run state", which is now the opposite for preflight's check.
  - Treat that sentence as MEDIUM-adjacent staleness. It is a doc contradiction in an unchanged line that the diff forces: the settle log is now exempt from the clean check, not counted by it.

**Tool calls**
1. Read `PATTERN-leaf-packages.md` and `PATTERN-fabric-isolation.md`, and grepped for preflight in the pattern files; the manifest lacked them.
2. Grepped the repository for other `worktree-clean`, `fabricengine.Clean` and `Clean(l` consumers to find callers and docs beyond the diff.
3. Ran `go list` on the `shedrun` and `preflight` imports to rule out a cycle and confirm the new edge.
4. Read `preflightshed/preflight.go:140-175` and grepped `fabricengine/doc.go` for `Clean` mentions, to see how the shed preflight reaches `Clean` and what the fabric docs say about it.
