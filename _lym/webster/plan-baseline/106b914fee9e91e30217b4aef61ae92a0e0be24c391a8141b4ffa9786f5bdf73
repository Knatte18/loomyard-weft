# Card 8 — rerun-mark-writer-only

**Create:**
- `plan:internal/shuttleengine#RerunMark` -> `type RerunMark struct`
- `plan:internal/shuttleengine#RerunMark.Run` -> `func (m *RerunMark) Run(gate Gate) (GateResult, error)`

**Edit:**
- `internal/shuttleengine#GateResult`
- `internal/shuttleengine#GateEntry`
- `internal/gatefixer#WrapRowGate`
- `internal/gatefixer#rowWrap`
- `internal/gatefixer#rowWrap.fixerEntry`
- `internal/gatefixer#rowWrap.spawning`
- `internal/gatefixer#rowWrap.spawnAndJudgeAgain`
- `internal/gatefixer#TestWrapRowGate`

**Extends:**
- `internal/shuttleengine/doc.go`
- `internal/gatefixer/doc.go`

**Uses:**
- `internal/gatefixer#newRowHarness`
- `internal/gatefixer#verifyEntry`

**Intent:**
Give the cold-fixer wrap the two hooks D3 needs, so a reach turn's result goes to the writer alone and a post-fixer re-run counts no failure.

- `GateResult` gains `WriterOnly bool`: a failing result meant for the writer alone, which no cold fixer is handed. The shed driver treats it as any failing result.
- `RerunMark` marks a gate call as a re-run: `Run(gate)` sets the mark, calls `gate`, clears the mark and returns its result.
  Card 9 adds `On()`, the mark's read, with the reach turn that first calls it.
  Its zero value is ready, and a nil `*RerunMark`'s `Run` calls the gate unmarked, so an entry with no mark is unchanged.
  It is safe for concurrent use.
- `GateEntry` gains `Rerun *RerunMark`, the mark a wrap sets while it runs the entry's gate again over a fixer's tree; nil for an entry nothing re-runs.
- `WrapRowGate` keeps every entry's `Rerun`. `rowWrap` keeps the fixer entry's mark (`fixerEntry`).
  `spawning` returns a failing result with `WriterOnly` set as it is, spawning no fixer, for both the entry's own gate and its forked gate;
  every other failure still goes to the fixer first, as today.
  `spawnAndJudgeAgain` runs the inner gate after the fixer through the mark's `Run`, so that second run is a re-run.
  The wait on a live record a restarted driver finds runs the gate unmarked: it is that arrival's only run.
  A `WriterOnly` result of the re-run is returned with the fixer's reply appended, as any re-run verdict is.
  Bound: the bypass lets through exactly the results their gate marks `WriterOnly`, which card 9's reach turn alone sets.

`TestWrapRowGate` (tier0) gains subtests on its `scriptedGate` harness:
a failing `WriterOnly` result of the entry's gate spawns no fixer and is returned as it is, and so does one of its forked gate;
a plain failure still spawns one fixer;
a plain failure whose re-run after the fixer returns a `WriterOnly` result returns that result to the writer with the fixer's reply appended, and spawns no second fixer;
an entry with a nil `Rerun` is wrapped and judged as before.

`shuttleengine/doc.go` gains `WriterOnly` and `RerunMark`; `gatefixer/doc.go` gains the bypass and the re-run mark.

**ImpactSummary:** Two additive fields and one small type in shuttleengine, whose zero values keep every existing gate and wrap as they are; gatefixer's wrap gains one skip and marks one call; only `TestWrapRowGate` changes.
