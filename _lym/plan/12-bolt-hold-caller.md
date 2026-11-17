# Card 12 — bolt-hold-caller

**Create:**
- `plan:internal/pauserecord#BoltHoldCaller` -> `const BoltHoldCaller = "bolt-hold"`

**Edit:**
- `internal/pauserecord#Entry`
- `internal/pauserecord#Coverage`
- `internal/pauserecord#Record.CoverageOf`
- `internal/pauserecord#CoveredError`
- `internal/pauserecord#heldLine`
- `internal/pauserecord#TestCoverageOf`
- `internal/pauserecord#TestCoveredError`
- `internal/pauserecord#TestWaitUncovered`
- `internal/pauserecord/doc.go`

**Uses:**
- `internal/pauserecord#OwnerCaller`

**Intent:** Add the pause record's `bolt-hold` caller, which holds a run's processes on a machine that reads a Bolt hold, and make every pause reader word it as a hold (D5).

`BoltHoldCaller` is the `Caller` of a slug entry a machine writes for one of its runs while a Bolt hold stands; it is apart from the legacy `HoldCaller` (`adopt-hold`) and `AdoptCaller`, which stay declared only for the upgrade's first read, and from `OwnerCaller`, which no code writes after card 23 and the upgrade still reads.
`Entry` gains `Machine string` (`json:"machine,omitempty"`): the hold's machine, set on a `bolt-hold` entry.
`CoverageOf` sets `Held` for a live entry of the slug's own scope carrying `BoltHoldCaller` too, and `Coverage` gains `HoldMachine`, that entry's machine.

`CoveredError` words a covering `bolt-hold` entry as held by an adopt on that machine:
`pause: an adopt on machine <m> holds this run since <time>; way forward: wait for that adopt's hold to clear, which lifts this entry at the next tick of the worktree's dobby (with no dobby running, `lym beam start` starts one), then <retry>; if machine <m> has stopped, ask the operator`.
It never names `lym tender resume` for such an entry, nor `--take-hold`.
`heldLine`, the line a wait writes when it starts, names the adopt and its machine the same way.
Every other entry's wording is unchanged, so `loomcli`'s and `shuttleengine`'s refusals, which call `CoveredError`, name the hold's machine with no change of their own.

Tests, tier0:
`TestCoverageOf` gains a `bolt-hold` row setting `Held` and `HoldMachine`.
`TestCoveredError` gains a `bolt-hold` row asserting the machine and the absence of `lym tender resume`.
`TestWaitUncovered` gains a row whose start line names the hold's machine.
`doc.go` describes the caller and its readers.

**ImpactSummary:** One new caller constant and one optional entry field; CoveredError's four callers and the shared wait gain the hold wording with no change of their own.
