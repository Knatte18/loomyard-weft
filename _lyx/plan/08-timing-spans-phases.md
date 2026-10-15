# Card 8 — timing-spans-phases

**Create:**
- `plan:internal/reportengine#Interval` -> `type Interval struct`
- `plan:internal/reportengine#TimingOptions` -> `type TimingOptions struct`
- `plan:internal/reportengine#TimingSelection` -> `type TimingSelection struct`
- `plan:internal/reportengine#TimingReport` -> `type TimingReport struct`
- `plan:internal/reportengine#TimingRun` -> `type TimingRun struct`
- `plan:internal/reportengine#PhaseMedian` -> `type PhaseMedian struct`
- `plan:internal/reportengine#TimingBucket` -> `type TimingBucket struct`
- `plan:internal/reportengine#BucketDay` -> `const BucketDay = "day"`
- `plan:internal/reportengine#BucketWeek` -> `const BucketWeek = "week"`
- `plan:internal/reportengine#PhaseSetup` -> `const PhaseSetup = "setup"`
- `plan:internal/reportengine#PhaseDiscussion` -> `const PhaseDiscussion = "discussion"`
- `plan:internal/reportengine#PhasePlan` -> `const PhasePlan = "plan"`
- `plan:internal/reportengine#PhaseWebster` -> `const PhaseWebster = "webster"`
- `plan:internal/reportengine#PhaseMerge` -> `const PhaseMerge = "merge"`
- `plan:internal/reportengine#PhasePublishFinalize` -> `const PhasePublishFinalize = "publish-finalize"`
- `plan:internal/reportengine#PhaseLand` -> `const PhaseLand = "land"`
- `plan:internal/reportengine#PhaseOther` -> `const PhaseOther = "other"`
- `plan:internal/reportengine#RunStateLanded` -> `const RunStateLanded = "landed"`
- `plan:internal/reportengine#RunStateInFlight` -> `const RunStateInFlight = "in-flight"`
- `plan:internal/reportengine#ComputeTiming` -> `func ComputeTiming(src Sources, opts TimingOptions) (TimingReport, error)`
- `plan:internal/reportengine#selectInFlight` -> `func selectInFlight(src Sources, opts TimingOptions, findings *[]Finding) ([]RunRow, error)`
- `plan:internal/reportengine#phaseOf` -> `func phaseOf(rows []timedRow, index int) (string, bool)`
- `plan:internal/reportengine#timedRow` -> `type timedRow struct`
- `plan:internal/reportengine#mergeRows` -> `func mergeRows(row RunRow) ([]timedRow, []Finding)`
- `plan:internal/reportengine#timeRun` -> `func timeRun(row RunRow, opts TimingOptions, snapshot time.Time) (TimingRun, []Finding)`
- `plan:internal/reportengine#unionIntervals` -> `func unionIntervals(set []Interval) []Interval`
- `plan:internal/reportengine#withoutIntervals` -> `func withoutIntervals(set, cut []Interval) []Interval`
- `plan:internal/reportengine#clipIntervals` -> `func clipIntervals(set []Interval, window Interval) []Interval`
- `plan:internal/reportengine#bucketWindow` -> `type bucketWindow struct`
- `plan:internal/reportengine#bucketWindows` -> `func bucketWindows(by string, window Interval) []bucketWindow`
- `plan:internal/reportengine#medianSeconds` -> `func medianSeconds(values []int64) *int64`
- `plan:internal/reportengine#world.pairBranch` -> `func (w *world) pairBranch(slug, sha string, when time.Time, seed, status string, extra map[string]string)`
- `plan:internal/reportengine#TestComputeTiming` -> `func TestComputeTiming(t *testing.T)`
- `internal/reportengine/timing_test.go`

**Edit:**
- `internal/reportengine#RunRow`
- `internal/reportengine#readLoomRecords` -> `func readLoomRecords(records Reader, tag gitrepo.TagRef, slug string) (seed shedrun.Seed, status shedengine.Status, reason string, findings []Finding, err error)`
- `internal/reportengine#selectRuns`
- `internal/reportengine#world`
- `internal/reportengine#world.sources`
- `internal/reportengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Add `ComputeTiming`, the timing report's compute entry beside `Compute` over the same `Sources` snapshot: which runs it covers, each run's span and per-row intervals, the phase each interval counts under, machine sleep, the day or week buckets and the phase medians.
Waits and attempts (card 9) and stops by settler (card 10) extend the run and the bucket; the markdown (card 11) and the verb (card 12) follow.
A record that does not decode is a `Finding`, and a figure a run's records lack is absent (nil), never zero, as in `convergence`.

Options and output.
`Interval` is `From`, `To time.Time`.
`TimingOptions` holds `Since` (the UTC date the window opens at), `By` (`BucketDay` or `BucketWeek`), `Live` (the slugs whose pair the caller found live), `Sleeps []Interval` and `SleepNote`, all told: the engine runs no process and reads no clock, its window closing at `Sources.SnapshotAt`.
`TimingReport` is `Selection TimingSelection` (`records_branch`, `records_tip`, `since`, `by`, `snapshot_at`), `Runs []TimingRun`, `Phases map[string]*PhaseMedian`, `Buckets []TimingBucket`, `SleepNote string` and `Findings []Finding`, with JSON keys `selection`, `runs`, `phases`, `buckets`, `sleep_note` and `findings`, and every slice non-nil.
`TimingRun` carries `slug`, `tag` (the archive tag, or the records branch's name for a run in flight), `state` (`RunStateLanded` or `RunStateInFlight`), `start`, `end`, `span_seconds`, `sleep_seconds`, `active_seconds` and `phases` (phase name to `*int64` seconds); every duration in JSON is integer seconds.
`TimingBucket` carries `label`, `start`, `runs` (how many runs' spans overlap it), `phases` (seconds), `sleep_seconds` and `active_seconds`.
`PhaseMedian` is `seconds *int64` and `runs int`, the landed runs holding the phase.

Run selection.
`ComputeTiming` calls `selectRuns` with `Options{Since: opts.Since}` into a scratch `Report`, so a landed run is exactly a full run as `convergence` selects it, with its seed, status, batten snapshots, parent-review rounds, arbiter attempts and stops read once; partial runs enter neither the run table nor any bucket, and the scratch report's findings join the timing findings.
`readLoomRecords` also returns the decoded seed, and `selectRuns` keeps its `started_at` on a new unexported `RunRow` field; `RunRow` also gains the in-flight mark and the loom status's current producer, for the in-flight path.
`selectInFlight` reads `Sources.Branches`, keeping only the pair records branches: a branch other than `Sources.RecordsBranch` whose name `fabricengine.WeftWarpSlug` parses, the slug being what it returns.
A branch whose tip is the commit of one of its slug's archive tags is landed and skipped without a finding; a branch whose slug `TimingOptions.Live` names is a run in flight; any other is abandoned, left out with a `Finding` naming the branch and saying it has no archive tag at its tip and no live pair.
A run in flight reads its seed and status at the branch tip through `shedrun.ParseSeed` and `shedengine.ParseStatusTolerant`: an absent seed or status is a pair not seeded yet, left out silently; a seed naming another recipe is left out silently, as `convergence` leaves such a tag out of its runs; a record that does not decode is a finding and the branch is left out.
It takes the batten snapshots `battenIndex.snapshotsOfRun` returns for the slug committed after the slug's newest archive tag and at or before `SnapshotAt`, so a reused slug never reads an earlier run's, and reads its other sources through the same `RunRow` methods `selectRuns` calls, at the tip named by the branch.
A run enters the report when its span overlaps the window from `Since` to `SnapshotAt`; a `Since` after the snapshot gives an empty report with no bucket.

Span and intervals (`mergeRows`, `timeRun`).
The span opens at the first entry of the run's batten record (the last snapshot), or at the seed's `started_at` without the batten source, or, when both lack a stamp, at the first loom row with a finding saying so.
`mergeRows` merges the loom history and the batten record's history in stamp order, each history keeping its own order, a row's stamp being its `last_at` when set, else its `at`; each row's interval runs from the latest merged stamp seen so far, the span's opening for the first row, to its own stamp, so a batten row between two loom rows splits their gap.
A stamp earlier than the latest stamp so far gives a zero interval and a finding naming the row and the file, and the next row's interval still starts from the latest stamp.
A row whose `at` is empty or does not parse as RFC 3339 has no stamp: it gives a zero interval and a finding naming the row and the file, never opens or closes the span, and leaves the latest stamp where it was.
The span closes at the last stamp of either history, or at `SnapshotAt` for a run in flight, whose trailing interval from its last stamp counts under the phase of its loom status's current producer.

Phases (`phaseOf`, given the merged rows and a row's index, each `timedRow` carrying its history, its producer, its outcome and its position in that history), one closed table: `setup` holds every batten row before batten's first `Run-Shed` entry and loom's `Preflight` and `Loom-Preflight`; `discussion` every `Discussion-*` row; `plan` every `Plan-*` row and `Batchifier`; `webster` every `Webster*` row and `Describe`; `merge` every `Merge-*` row; `publish-finalize` `Publish`, `PR-Align`, `PR-Gate`, `PR-Rework`, every `Darn-*` row, `Finalize` and `Friction-Reflect`; `land` every batten row other than `Run-Shed` after batten's first `Run-Shed` entry.
The batten rows are placed by position, split at the first `Run-Shed` entry into `setup` before it and `land` after it, as the decision record's table states them; the loom names come from `loomshed`'s constants and `Run-Shed` from `battenrecipe`'s, never literals.
Batten's `Run-Shed` entries record the child's state changes, and the table names no phase for them; counting each under `other` with a finding would put a finding on every run, so each counts under the phase the child was in: the phase of the producer its change names (`battenshed.ParseStateChange`), else the phase of the latest loom row before it, else `setup`.
Any other row counts under `other` and adds a finding naming it.
Without the batten source, `land` is nil and `setup` holds the loom rows alone.

Sleep.
`opts.Sleeps`, unioned and clipped to the span, is the run's `sleep_seconds`, and every row interval loses the part a sleep covers, so the phases, `other` included, plus the sleep sum to the span; `active_seconds` is the span less the sleep here, and card 9 subtracts the waits.

Buckets and medians.
`bucketWindows` cuts the window into UTC days labelled `2026-10-15`, or ISO weeks labelled as `isoWeekLabel` does, with Monday first; no bucket opens before `Since`.
Each interval and sleep is clipped to the window and to each bucket it overlaps, so an interval spanning UTC midnight splits across both days, and a bucket's time sums over runs, so two parallel runs add two hours to an hour.
`Phases` holds each phase's median, through `medianSeconds`, over the landed runs holding it, only those whose archive tag is committed in the window; a run in flight never enters a median, and a phase no landed run holds has nil seconds.
The median of an even count is the mean of the two middle values, rounded down to a whole second.

Test fixture.
`world` gains its branches: `world.pairBranch` commits a pair's records tip off the main line holding the given seed, status and extra files and records a `gitrepo.BranchRef` named `fabricengine.RecordsBranchName(slug)`, and `world.sources` returns them, with a `BranchRef` for the records branch itself, as `Branches`, with a fixed `SnapshotAt`.

`TestComputeTiming` in `internal/reportengine/timing_test.go` is one table; this card's rows: a landed run with both histories, whose phase seconds, `other` included, plus sleep sum to its span, and whose batten row between two loom rows splits their gap at the asserted seconds; a folded entry whose `last_at` closes its interval; an out-of-order stamp giving a zero interval and a finding, the next interval starting from the latest stamp; an unknown row counted under `other` with a finding; a run without the batten source opening at the seed's `started_at` with `land` nil; a run with neither stamp opening at its first loom row with a finding; a sleep inside a row counted as sleep and not in the row's phase; an interval across UTC midnight split over two day buckets, and the same run under `BucketWeek` in its ISO week; a row with an empty `at` giving a zero interval and a finding while the span opens and closes at the stamped rows; a landed run whose histories hold one row of every table entry, batten rows before, between and after its `Run-Shed` entries included, each counted under its phase, the row between two `Run-Shed` entries under `land`; three `Run-Shed` entries, one whose change names a producer, one naming none after a loom row and one naming none before any loom row, counted under that producer's phase, the loom row's phase and `setup`; a run in flight in the run table and its buckets, absent from every median, its last interval running to `SnapshotAt` under the phase of its status's current producer; a pair branch whose tip is its slug's archive tag commit counted once, as the landed run, with no finding; the records branch itself and a branch `WeftWarpSlug` does not parse left out with no finding; a live pair whose branch holds no seed left out with no finding; a live pair whose seed names another recipe left out with no finding; a live pair whose status does not decode left out with a finding; two landed runs whose phase median is the rounded-down mean of their two values; a branch neither tagged nor live left out with a finding; a live slug reused after a tag reading only the batten snapshots after that tag; two overlapping runs whose bucket holds the sum of both; a partial run in neither the run table nor any bucket; a run starting before `Since` whose bucket list starts at `Since`; and a `Since` after the snapshot giving no run and no bucket.

`internal/reportengine/doc.go` gains a "Timing" section stating the selection, the span, the merged intervals, the phase table, sleep, buckets and medians as above, and its "One ref snapshot" paragraph names `Branches` and `SnapshotAt`.
`cmd/lyx/testdata/test-budget.yaml` raises `internal/reportengine` to 9.

**ImpactSummary:** convergence's selection and figures are untouched; readLoomRecords' one caller, selectRuns, is on this card, and RunRow gains unexported fields only, so its JSON is unchanged.

**Commit:** 8: Compute run time per phase, sleep and bucket for the timing report
