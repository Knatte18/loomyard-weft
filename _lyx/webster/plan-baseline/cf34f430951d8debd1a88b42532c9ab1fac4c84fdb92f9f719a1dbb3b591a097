# Card 9 — launch-list

**Edit:**
- `internal/gatecli#goTestEvent`
- `internal/gatecli#goJSONCounter`
- `internal/gatecli#goJSONCounter.handle`
- `internal/gatecli#counter`
- `internal/gatecli#vstestCounter`
- `internal/gatecli#tierResult`
- `internal/gatecli#runTier`
- `internal/gateslot#RunRecord`
- `internal/gatecli#TestCounters`
- `internal/gateslot#TestWriteRunRecord`

**Create:**
- `internal/gateslot#PackageLaunch`
- `internal/gatecli#goJSONCounter.Launches`
- `internal/gatecli#vstestCounter.Launches`

**Extends:**
- `internal/gatecli/doc.go`
- `internal/gateslot/doc.go`

**Intent:**
A go-json tier run records per package how long its test binary took to launch (D11).

`goTestEvent` also decodes `Package`, `Time` and `Elapsed`.
On package-level events (no `Test`), `goJSONCounter.handle` keeps, per package, the time of its `start` event, the gap from it to the package's next event, and the elapsed seconds of its closing `pass` or `fail` event; a package whose output reports `(cached)` or `[no test files]` is left out, since its binary did not run.
The rendered log stays the events' output only.
The `counter` interface gains `Launches`, which the vstest counter answers with nil; `runTier` puts the list on the record.
`PackageLaunch` holds `package`, `launch_seconds` and `elapsed_seconds`, and `RunRecord` gains `Launches` (`launches`, omitted when empty), read tolerantly, so a record written before this task carries none.
The gap holds the binary's exec, any host-scanner hold such as Cortex XDR's, its init and its `TestMain` setup; `go doc cmd/test2json` defines `start` as the test binary about to be executed.

`TestCounters` gains rows: a package with `start`, a later event and a closing `pass` with `Elapsed` gets one launch entry with the gap and the elapsed time; a `(cached)` package and a `[no test files]` package get none; the text output is unchanged.
`TestWriteRunRecord` round-trips the list and reads a record without it.

**ImpactSummary:** Adds the launch list to the go-json counter, the counter interface and the run record; vstest and every reader of the text output are unchanged.
