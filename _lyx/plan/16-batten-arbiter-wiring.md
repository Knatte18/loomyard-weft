# Card 16 — batten-arbiter-wiring

**Create:**
- `plan:internal/battencli#arbiterInFlightFrom` -> `func arbiterInFlightFrom(present bool, readMarker func() (loomengine.ArbiterMarker, bool, error), directory func() ([]reedengine.DirectoryRow, error)) (battenshed.ChildArbiter, bool, error)`
- `plan:internal/battencli#stopDeadArbiterStrands` -> `func stopDeadArbiterStrands(rows []reedengine.DirectoryRow, stop func(guid string) error) error`
- `plan:internal/battencli#TestArbiterInFlightFrom` -> `func TestArbiterInFlightFrom(t *testing.T)`
- `plan:internal/battencli#TestStopDeadArbiterStrands` -> `func TestStopDeadArbiterStrands(t *testing.T)`

**Edit:**
- `internal/battencli#battenCLI.wire`
- `internal/battencli/cli.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/battenshed#ChildArbiter`
- `plan:internal/loomengine#ReadArbiterMarker`
- `plan:internal/loomengine#IsArbiterStrand`
- `plan:internal/shedrun#ArbiterMarker`

**Intent:**
Wire the batten hold to the child's marker (D10) and keep batten's strand revive from relaunching an arbiter strand (D11).

`arbiterInFlightFrom`, beside `driverStrandFrom` in `wire.go`, reads the marker through `readMarker`; an absent task worktree or an absent marker is not found; otherwise it reads the pair's reed directory and reports the marker's deadline with `Alive` true only when a live row carries the marker's strand guid.
A directory with no reed session reads as not alive, as `driverAliveFrom` treats it.

`stopDeadArbiterStrands` calls `stop` for every row that `loomengine.IsArbiterStrand` names and that is not live, and returns the first error.
A live arbiter row is left alone: it keeps working and its verdict waits for the revived driver.

In `battenCLI.wire`, `ArbiterInFlight` is the closure over `arbiterInFlightFrom`, reading `loomengine.ReadArbiterMarker(shedrun.ArbiterMarker(taskLocation, shedrun.SelfRunID))` and the task pair's reed directory as `DriverStrand` does.
`ReviveStrands` first lists the pair's reed directory and passes it to `stopDeadArbiterStrands` with a stop that goes through shuttle's stop verb, `StopStrand` of a `shuttleengine.Runner` over the task pair (`PATTERN-shuttle-stop`), which settles the run record and removes the strand; only then does it run reed's `Resume`.
A stop error fails the revive, which the Inner-Run row already reports with its fallback.
Bound: a dead arbiter strand is ended rather than relaunched; the arbiter verb, or a re-run of it, reads that attempt as `hand-on`.

`TestArbiterInFlightFrom` is table-driven: absent worktree, absent marker, a marker whose strand row is live, dead, or missing, and a directory error returned.
`TestStopDeadArbiterStrands` is table-driven: dead arbiter rows stopped, live arbiter and driver rows untouched, a numbered `arbiter-2` row matched, and a stop error returned.
Raise `internal/battencli`'s row in the test budget by two.
Name the hold's wiring and the revive's arbiter rule in the package doc in `cli.go`.

**ImpactSummary:** batten's Inner-Run deps gain one reader and its revive one pre-step; no other batten path changes.
