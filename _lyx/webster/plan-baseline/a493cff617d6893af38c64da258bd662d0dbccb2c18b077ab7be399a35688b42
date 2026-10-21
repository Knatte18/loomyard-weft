# Card 7 — gate-counters

**Create:**
- `plan:internal/gatecli#tierCounts` -> `type tierCounts struct`
- `plan:internal/gatecli#counter` -> `type counter interface`
- `plan:internal/gatecli#newCounter` -> `func newCounter(kind string, out io.Writer) (counter, error)`
- `plan:internal/gatecli#goJSONCounter` -> `type goJSONCounter struct`
- `plan:internal/gatecli#vstestCounter` -> `type vstestCounter struct`
- `plan:internal/gatecli#TestCounters` -> `func TestCounters(t *testing.T)`

**Intent:**
Name the two counters a tier declares (D7), so every tier run reports what it discovered and a Go tier's `-json` output reads as text.

A `counter` is an `io.Writer` the tier command's stdout and stderr go through, with `Flush() error` for a trailing partial line and `Counts() tierCounts` (discovered, passed, failed, skipped).
`newCounter("go-json", out)` returns a `goJSONCounter`: it splits its input into lines, decodes each as a `go test -json` event, writes each event's `Output` text to `out` in order, so failure lines, panics and build errors read exactly as plain `go test` prints them, and passes a line that is not a JSON event through verbatim, as `go build` and `go vet` steps of a tier write.
It counts test-level events only (a non-empty `Test`): each `run` is discovered, and each `pass`, `fail` and `skip` adds to its count; package-level events add nothing, so a package with only a `TestMain` or only benchmarks discovers zero.
`newCounter("vstest", out)` returns a `vstestCounter`: it passes every line through and sums the `Total`, `Passed`, `Failed` and `Skipped` fields of every VSTest summary line (`Passed!  - Failed:     0, Passed:    12, Skipped:     0, Total:    12, Duration: ...`, and its `Failed!` form) into discovered, passed, failed and skipped.
Any other kind is an error; `gateslot`'s loader already refuses it (card 2), so the error is a fail-loud wiring guard.
The counts decide nothing: a tier run's outcome stays its command's exit code (card 8), and zero discovered passes, as the operator decided after Discussion.

Tests: the new `TestCounters` table covers, through the public writer: a passing and a failing test, a skipped test, subtests counted each, a panic's output and a build error's non-JSON lines passed through as text, a line split across two writes, a trailing line with no newline emitted by `Flush`, a `TestMain`-only package and a Benchmark-only package each discovering zero, two VSTest summary lines summed, a `Failed!` summary, and the unknown-kind error.
Bound: the parse reads only the event shapes `go test -json` and VSTest print; a VSTest version printing another summary shape counts zero, visible in the log (decision record, open risks).
