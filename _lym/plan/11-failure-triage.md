# Card 11 — failure-triage

**Create:**
- `plan:internal/gateslot#TriageLinePrefix` -> `const TriageLinePrefix = "triage: "`
- `plan:internal/gateslot#TriageLines` -> `func TriageLines(log string) map[string][]string`
- `plan:internal/gateslot#TestTriageLines` -> `func TestTriageLines(t *testing.T)`
- `plan:internal/gatecli#triageLookup` -> `type triageLookup struct`
- `plan:internal/gatecli#newTriageLookup` -> `func newTriageLookup(open func() (*testreach.Reader, error), mergeBase func() (string, bool, error), changed func(base string) ([]testreach.ChangedFunc, error)) (triageLookup, bool)`
- `plan:internal/gatecli#triageLookup.lines` -> `func (l triageLookup) lines(pkg, test string) []string`
- `plan:internal/gatecli#TestTriageLookup_Lines` -> `func TestTriageLookup_Lines(t *testing.T)`

**Edit:**
- `internal/gatecli#runFailure`
- `internal/gatecli#failedRun.reportLines`
- `internal/gatecli#renderFailureReport`
- `internal/gatecli#execution.printReport`
- `internal/gatecli#TestRenderFailureReport`
- `internal/gatecli#TestGateTest_Scenario`
- `internal/gateslot#TestFailedTests`

**Extends:**
- `internal/gatecli/doc.go`
- `internal/gateslot/doc.go`

**Uses:**
- `plan:internal/testreach#Open`
- `plan:internal/testreach#Reader.FuncsReachedBy`
- `plan:internal/gatecli#changedFuncsAt`
- `internal/gatecli#parentMergeBase`
- `internal/gatecli#redundancyModule`
- `internal/gatecli#TestRunWithFixer`
- `internal/gateslot#FailedTests`
- `internal/hubgeom#IsPrime`

**Intent:**
Add D7's triage lines to the failure report of a failing `lym gate test` call.

`execution.printReport` builds a `triageLookup` once per report, when the call is in a hub, its scoper is `go` and a run failed a test, through `newTriageLookup`:
- `open` opens `testreach.Open` over the shared maps and, outside the prime, the anchor maps; an error from it builds no lookup, so the report gets no triage line anywhere (D7's failed lookup).
- `mergeBase` reads the merge base with the parent branch (`parentMergeBase`); in the prime, which has no parent branch, it answers HEAD, as `touchedFiles` does.
  A task worktree whose read finds no parent branch, and one whose read fails, give a lookup with no change set and that reason.
- `changed` takes the run's change set, `changedFuncsAt` from the base to the working tree.
It then fills each failed test's `runFailure.triage` from `triageLookup.lines(pkg, test)` before rendering.

`triageLookup.lines(pkg, test)` looks a failed test up by its top-level name (a subtest path is cut at its first `/`) and returns the lines, each opening with `gateslot.TriageLinePrefix`:
- a test a map holds, with a change set: one line naming the changed functions it reaches, at most five then the count of the rest, and the map's commit;
  or, reaching none, one line saying "reaches none of this run's changes" with the map's commit, followed by a second line holding the D8 command
  `lym gate reach --test <package>.<Test> --since <last green commit>` for the caller to fill, on its own line so the line byte cap never cuts it;
- a test a map holds, with no change set: one line saying no change set exists and why (no parent branch, or the merge base read failed), with the map's commit, and no D8 command;
- a test no map holds: one line saying "not in the coverage maps".
The caps and phrases are named constants.

Rendering: `failedRun.reportLines` puts a failure's triage lines first under its two-space test line, as four-space lines, so the report's parsers read them as the test's output.
`renderFailureReport` counts the report's lines without triage first; it then adds each failure's triage lines, in report order, only while the line budget still holds them whole,
so a triage line never causes a cut and triage lines are dropped before any test output line is cut.

`gateslot.TriageLines(log)` is the failure report's reader of those lines (gateslot already parses the report for `FailedTests`):
it returns, per failed test as `<package>.<Test>` (the two-space test line's identity), the four-space lines under it that open with `TriageLinePrefix`, with the prefix kept; a log holding no report returns an empty map.
`FailedTests` needs no change: it already skips four-space lines.

The card fixer's report is the call's stderr copied verbatim (`detachRun.fixAndRerun`), which `TestRunWithFixer` pins;
with `TestGateTest_Scenario` pinning the triage line in that stderr's failure report, the card fixer's report carries the lines.

Tests:
- `TestTriageLookup_Lines` (tier0) is one table over lookups `newTriageLookup` builds on maps under `t.TempDir()` with scripted `mergeBase` and `changed`:
  a test reaching two changed functions; one reaching six (five named and the count); one reaching none, with its command line; a subtest looked up by its top-level test;
  a test no map holds; no parent branch, and a failed merge-base read, each with its reason and no command line; an `open` that fails builds no lookup.
  Each line naming a mapped test asserts the map's commit in it.
- `TestRenderFailureReport` (tier0) gains rows: a failed test with triage lines renders them first under its test line; a report at the line cap keeps every test output line and drops triage lines first.
- `TestFailedTests` (tier0) gains one row: a report holding triage lines parses to the same failed tests as without them.
- `TestTriageLines` (tier0) is one table: a report with triage lines under two tests, a report with none, and a log with no report.
- `TestGateTest_Scenario` (tier1): its subtest whose go tier fails `TestFails` in a hub with scoper `go` asserts the failure report's first line under the failed test is the "not in the coverage maps" triage line, since no map holds it;
  this row reaches `printReport`'s wiring of the lookup.

`gatecli/doc.go`'s `# Failure report` section gains the triage lines and their budget rule; `gateslot/doc.go` gains `TriageLinePrefix` and `TriageLines`.
Bound: the lines are advisory; a stored map may lack a function the run added, and a stale entry can show a reach the test no longer has.

**ImpactSummary:** The failure report gains advisory four-space lines under failed tests that the budget drops first; `FailedTests` and the reruns it drives parse the report unchanged, which the new row pins; `TestGateTest_Scenario`'s failure-report assertion gains the triage line, and the card fixer's report, the call's stderr, carries it.
