```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the fix covers every member of the set-shaped findings (F7, F8, F9).

Unchecked findings: none. F1 and F6 are decision-class and not the fixer's.

No findings.

- F7: f4fd75355's deleted and trimmed tests each have a trailer, an existing trailer, or a holder in another file. In earlier commits, the net-removed assertions I sampled were moved, trailered, or held in another file. I did not compare moved tests line by line.
- F8: the named sites are rewritten. The comments still mention `Fabric.Commit`, `Fabric.Pull` and `fabriccli`, but these are exported or public surfaces, outside the finding's categories (engine-private functions, an engine file, engine error types).
- F9: all four branches have a case (repeated phase, nil lock, blob tag under both backends, prunable records worktree).

Tool calls:
1. Listed net-removed assertion lines per commit before f4fd75355, because my context held only those commit bodies and the f4fd75355 diff.
2. Read the old `TestIsReservedHubName` subtests at `e31750e43~1`, because my context showed only their names and the new table.
3. Searched the `fabriclayout` comments for engine references, because my context held only `mutation.go` and `verbresults.go` and an identifier-based search.
