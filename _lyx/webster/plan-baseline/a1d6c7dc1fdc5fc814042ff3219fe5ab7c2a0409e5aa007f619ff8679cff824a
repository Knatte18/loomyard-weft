# Card 14 — shedrun-cleanup-paths

**Create:**
- `internal/shedrun#RunsRoot`
- `internal/shedrun#ScratchRoot`
- `internal/shedrun#RunDirRel`

**Edit:**
- `internal/shedrun#TestRunsRootRel`
- `internal/shedrun#TestRunDirResolution`
- `internal/shedrun/doc.go`

**Intent:**
Add the three path constructors `lyx batten cleanup` needs, so it names no `shed` segment itself (`PATTERN-shed-run-directory`).
`RunsRoot(l)` is the absolute durable directory every `RunDir` sits under, `l.AnchorPath()` joined with `RunsRootRel()`;
`ScratchRoot(l)` is its ephemeral mirror, `l.AnchorPath()` joined with `lyxdirs.DotLyxDirName` and the `shed` segment, the directory every `ScratchDir` sits under and the one `PrimeRunLock` and `LandLock` also live in;
`RunDirRel(l, runID)` is `RunDir`'s anchor-relative form, with the same `runSegment` alias resolution `StatusRel` applies, the pathspec a records commit of a whole run directory's deletion takes.
`TestRunsRootRel` also asserts `RunsRoot(l)` and `ScratchRoot(l)` equal `filepath.Dir` of `RunDir(l, id)` and `ScratchDir(l, id)` for a plain run-id;
`TestRunDirResolution` also asserts, for each of its rows, that `filepath.Join(l.AnchorPath(), RunDirRel(l, id))` equals `RunDir(l, id)`.
`internal/shedrun/doc.go` lists the three constructors with the others.

**ImpactSummary:** Three new exported constructors and two extended tests; no existing constructor changes.
