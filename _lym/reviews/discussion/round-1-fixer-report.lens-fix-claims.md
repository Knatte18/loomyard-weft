```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added or rewrote holds against the code at this worktree's HEAD.

no findings

Checked against the code:
- **D2, testmap's skip:** testmap already skips a package that holds a dirty Go file (`internal/gatecli/testmap.go:363-364`). Its hashes are read at HEAD through `CodeFileAt` (`internal/gatecli/testmap.go:436`), so "the content at its `Commit`" holds.
- **D3 Bound:** `internal/testcover/spawnscan.go:257-260` returns an empty result for a call through an import name. The `ScanSpawns` doc (lines 171-174) names the testkit following, the empty tier0 verdict and the hook and alias blind spot.
- **Constraints, PATTERN-tolerant-records:** `WriteMap` keeps undeclared keys (`internal/testcover/coveragemap.go:68`). An older binary rewrites the declared `Commit` and `BuiltAt` but keeps the old marker, so a mismatch shows.
- **Constraints, PATTERN-fabric-boundary:** testmap's hashes use `CodeFileAt` (`internal/gatecli/testmap.go:436`). `mapDrift` uses `commitTree`, which calls `CodeFileAt` (`internal/gatecli/redundancy.go:350`, `internal/gatecli/redundancytests.go:73`).
- **Notes, kept-set hint:** the kept set is built from `p.m.Blocks[index]` keys (`internal/gatecli/redundancy.go:592-593`).
- **Measured cost split:** I did not recompute it. It is the reviewer's measurement, carried over with its source named.

Tool calls:
- grep of `internal/gatecli/testmap.go`, `internal/gatecli/redundancytests.go` and `internal/gatecli/testmappackage.go`: whether testmap skips dirty packages, where its hashes come from, and which reader map drift uses. My context had only the function names, not their bodies.
- grep of `mapDrift` and the tree constructors in `internal/gatecli`: whether map drift reads through `CodeFileAt`, which the new constraint claims.
- ls and find under `_bolt/.lym`: where the gate run records are, for the measured-split claim. I did not go on to recompute the sums.
