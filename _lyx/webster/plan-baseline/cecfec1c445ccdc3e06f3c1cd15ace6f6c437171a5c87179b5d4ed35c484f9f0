# Card 15 — empty-tier-fails

**Create:**
- `internal/gatecli#emptyTierFailure`
- `internal/gatecli#TestEmptyTierFailure`
- `internal/gatecli#goJSONCounter.TestsRun`
- `internal/gatecli#vstestCounter.TestsRun`

**Edit:**
- `internal/gatecli#goJSONCounter`
- `internal/gatecli#goJSONCounter.handle`
- `internal/gatecli#counter`
- `internal/gatecli#vstestCounter`
- `internal/gatecli#runTier`
- `internal/gatecli#TestCounters`
- `internal/gatecli#TestGateTest_Scenario`
- `internal/gatecli/doc.go`

**Intent:**
Under the `go` scoper a tier run whose command lacks its tag fails instead of passing vacuously (D7).

The go-json counter keeps, per package, the number of tests it saw run, counted from the go-json events, so a package whose untagged `TestMain` still runs counts as zero; the `counter` interface gains `TestsRun`, which the vstest counter answers with nil.
After a passing go-json tier run under the `go` scoper, `emptyTierFailure` takes each package the events named with zero tests run, maps it to its directory under the module the run's directory holds, and scans its `_test.go` files with `go/parser`: a package holding a `Test` function in a file whose constraint holds under the tier's tag (`tierN`) on the host's GOOS and GOARCH fails the tier.
The failure exits 1 and its line, printed to stderr and the log before the summary line, names the tier, the package and `lyx config gate`, since the tier's command lacks its tag.
A run whose args after `--` carry `-run`, `-skip`, `-list` or `-bench` is exempt, since its filter may match nothing in a package; the guard step's runs are not exempt, since each package of a guard group holds a guard test its filter names.
`emptyTierFailure`'s `args` is the call's own words after `--` (`testRequest.args`), never the run's `args`, into which the guard step appends its `-run` filter, so that filter never exempts a guard run.
Bound: a filtered run under a command that lacks its tag passes vacuously; the site gates, which pass no args, catch it.
A hub without tier tags never meets the check, since no file then names a `tierN` tag.
The gatecli doc's Counters, record and log section is rewritten to the D7 rule: the sentence saying the counts decide nothing about the outcome, and the sentence saying a tier that discovers no tests passes with zero counted as a package with only a `TestMain` does, both change to say a go-json tier under the `go` scoper fails when a package holding a test under the tier's tag ran none.

`TestEmptyTierFailure` drives `emptyTierFailure` over fixture packages in a temp module: a package with a `tier1` test file and zero tests run fails naming the tier and package; a `tier1` test file constrained to another GOOS does not; a `-run` among the call's args exempts the run; a guard-step run, whose run args hold its `-run` filter while the call's args are empty, is not exempt and fails; a package that ran a test passes.
`TestCounters` gains a row for the per-package counts, including a package whose events come from an untagged `TestMain` alone (a `start`, its output and a closing `pass` with no test event), which counts zero.
`TestGateTest_Scenario` gains a row where a tier whose command lacks `-tags tier1` fails on a package holding a `tier1` test file beside an untagged `TestMain` file.

**ImpactSummary:** Adds a post-run check that can fail a go-json tier, and per-package counts on the counter interface; vstest tiers and filtered runs are unaffected.
