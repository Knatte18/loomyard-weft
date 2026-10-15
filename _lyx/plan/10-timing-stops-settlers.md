# Card 10 — timing-stops-settlers

**Create:**
- `plan:internal/reportengine#TimingStop` -> `type TimingStop struct`
- `plan:internal/reportengine#StopKindStop` -> `const StopKindStop = "stop"`
- `plan:internal/reportengine#StopKindBounce` -> `const StopKindBounce = "bounce"`
- `plan:internal/reportengine#StopKindBattenHalt` -> `const StopKindBattenHalt = "batten-halt"`
- `plan:internal/reportengine#SettledByJudge` -> `const SettledByJudge = "judge"`
- `plan:internal/reportengine#SettledByShed` -> `const SettledByShed = "shed"`
- `plan:internal/reportengine#SettledByTackler` -> `const SettledByTackler = "tackler"`
- `plan:internal/reportengine#SettledByArbiter` -> `const SettledByArbiter = "arbiter"`
- `plan:internal/reportengine#SettledByAlignment` -> `const SettledByAlignment = "alignment"`
- `plan:internal/reportengine#SettledByUnknown` -> `const SettledByUnknown = "unknown"`
- `plan:internal/reportengine#SettledByOpen` -> `const SettledByOpen = "open"`
- `plan:internal/reportengine#RunRow.readSettles` -> `func (row *RunRow) readSettles(records Reader, tag gitrepo.TagRef) ([]Finding, error)`
- `plan:internal/reportengine#runStops` -> `func runStops(row RunRow, rows []timedRow) ([]TimingStop, []Finding)`
- `plan:internal/reportengine#matchChildRecords` -> `func matchChildRecords(stops []TimingStop, row RunRow) (map[int]shedrun.SettleRecord, []Finding)`
- `plan:internal/reportengine#matchBattenRecords` -> `func matchBattenRecords(stops []TimingStop, row RunRow) (map[int]shedrun.SettleRecord, []Finding)`
- `plan:internal/reportengine#settlerOf` -> `func settlerOf(stop TimingStop, record *shedrun.SettleRecord, row RunRow, rows []timedRow) string`
- `plan:internal/reportengine#world.battenSettles` -> `func (w *world) battenSettles(sha string, hour int, slug string, status shedengine.Status, records ...shedrun.SettleRecord)`

**Edit:**
- `internal/reportengine#RunRow`
- `internal/reportengine#battenIndex.snapshotsOfRun` -> `func (b *battenIndex) snapshotsOfRun(slug, tag string, earlier *time.Time, until time.Time) ([]shedengine.Status, string, []Finding, error)`
- `internal/reportengine#selectRuns`
- `internal/reportengine#selectInFlight`
- `internal/reportengine#ComputeTiming`
- `internal/reportengine#TimingRun`
- `internal/reportengine#TimingBucket`
- `internal/reportengine#timeRun`
- `internal/reportengine#TestComputeTiming`
- `internal/reportengine/doc.go`

**Intent:**
Add each run's stops and who settled each one, read from the history labels of cards 5 and 6 and the settle records card 3 made decodable here.

Stops (`runStops`), three kinds counted apart:
`StopKindStop`, a halt or awaiting stop as `convergence` lists it, so this kind's count matches `convergence`'s: `RunRow.Stops` when the halt-note source is present, the k-th stop with a change taking the time of the k-th batten change into blocked, failed or awaiting, and a note-only stop, which carries no time of its own, counted in the bucket of the run's close; without that source `convergence` lists no stop, so this kind is absent (nil) and its settler counts are left out;
`StopKindBounce`, a loom `stuck` entry an `OnStuck` edge bounced (its `routed_to` set);
`StopKindBattenHalt`, a batten-row halt as `battenRowHalts` (card 9) finds it.
`TimingStop` carries `kind`, `producer`, `state`, `at` (null for a note-only stop) and `settler`.

Settle records.
`RunRow.readSettles` reads two settle-record files through `shedrun.ParseSettleRecords`, each under its own run directory at `shedrun.TreeRunRels(slug).SettleRecords`: the loom run's at the tag, or at the pair branch tip for a run in flight, and batten's at the commit holding the run's batten record, which `snapshotsOfRun` now also returns and `selectRuns` and `selectInFlight` keep on `RunRow`.
`ComputeTiming` calls it for landed runs and `selectInFlight` for runs in flight, never `selectRuns`, so `convergence` reads no settle file and its findings are unchanged.
An absent file holds no record; a line that does not decode is a finding naming the file and the line.
A record not committed at that tip simply is not there, and its stop reads `open` rather than adding a finding.
A record's stop key holds the history length the driven history had when the driver saw the halt, so its last counted entry is the entry that halted.
`matchBattenRecords`: a batten-file record names a batten-row halt when its key's producer and state equal the halt's and its history length is the 1-based position of the halt's entry in batten's history; batten-file records name nothing else.
`matchChildRecords`: a loom-file record names a child stop with a change when its key's producer and state equal the stop's and its history length is the greatest among such records whose last counted loom entry is stamped at or before the change, so a halt the Go driver restepped into a second halt before batten observed it reads the second halt's record, and the earlier record of the same producer and state folds into that stop with no finding.
A note-only stop takes the earliest record with its producer and state that no other stop took, and, when none is left, its settler comes from the later items.
A change whose producer is empty matches no record and falls to the later items.
Any other record matching no stop is a finding naming the file and the key.

Settlers (`settlerOf`), the first that holds:
`SettledByJudge` for a bounce;
`SettledByShed` when a batten automatic step accounts for the stop (an `AutomaticStep` with the change's producer and time) or its matched record has settler `go-rule` and action `restep` or `verify-retry`;
`SettledByTackler` when its matched record has settler `tackler` and action `restep`;
`SettledByArbiter` when an applied arbiter attempt accounts for it: its stop key's producer and state equal the stop's and its `spawned_at` is after the stop;
`SettledByAlignment` when an applied alignment verdict accounts for it: an applied PR-Gate alignment attempt spawned after a stop at `PR-Gate`, or an applied parent-review alignment record spawned after the stop;
the `by` of the first entry after the stop in the history that moved it on (the loom history for a child stop, batten's for a batten-row halt), the goto or the resumed entry: its own value (`orch`, `driver`, `operator` or another role), except that `shed` (`shedengine.CallerShed`) reads as `SettledByShed`, so a move by the Go driver or batten that no settle record covers reads `shed`;
`SettledByUnknown` when that entry carries no `by`, as every entry written before this task;
`SettledByOpen` when no entry follows, in a run in flight.
A `go-rule` record with action `moved`, `busy-exit`, `running-exit` or `hand-over`, a `spawn` record of any settler, a `parent` `hand-back` record and every `arbiter` record settle nothing by themselves and leave the stop to the later items; an arbiter settle is read from its applied attempt.

`TimingRun` gains `stops`; `TimingBucket` gains `settlers`, stop kind to settler to count, each stop counted in the bucket its time falls in, a note-only stop in its run's closing bucket.

`TestComputeTiming` gains rows reaching every settler class: a bounce reads `judge`; a halt a batten verify-retry automatic step accounts for reads `shed`; a loom-file `go-rule` `restep` record whose key matches a batten child change reads `shed`; a `tackler` `restep` record reads `tackler`; an applied arbiter attempt reads `arbiter`; an applied PR-Gate alignment attempt reads `alignment`; a stop whose only `go-rule` record is `moved`, followed by a goto with `by` `orch`, reads `orch`; a goto with `by` `shed` and no settle record reads `shed`; a resume entry with no `by` reads `unknown`; a stop in a run in flight with no later entry reads `open` with no finding; a halt restepped into a second halt before batten observed it reads the second record's settler and the first record adds no finding; a note-only stop takes its settler from a loom-file record with its producer and state; a change with an empty producer reads its settler from its later entry's `by`; a batten-file record names only a batten-row halt, counted under `batten-halt` and not among the `stop` kind, whose count equals `convergence`'s halt count over the same run; a loom-file record matching no stop is a finding; and an undecodable settle line is a finding.
Further rows reach the remaining branches: a halt whose matched record is `go-rule` `verify-retry` reads `shed`; stops whose only records are `go-rule` `busy-exit`, `running-exit` and `hand-over`, a `tackler` `spawn`, a `parent` `hand-back` and an `arbiter` record, each followed by a goto with `by` `operator`, read `operator`; an applied parent-review alignment record spawned after a stop reads `alignment`; a note-only stop with no unclaimed record of its producer and state reads its settler from its later entry's `by`; a batten-row halt followed by a batten goto with `by` `orch` and no batten-file record reads `orch`; a batten-file record matching no batten-row halt is a finding; a run without the halt-note source has a nil `stop` kind while its bounces and batten-row halts still count; and a note-only stop is counted in the bucket of its run's close.
`world.battenSettles` commits batten's status and its `go-settle.jsonl` together on the records main line, so the batten-file rows read both at one commit.
`internal/reportengine/doc.go`'s Timing section gains the stops, the record join and the settler order as above.

**ImpactSummary:** snapshotsOfRun's two callers are on this card and convergence ignores the new commit result; settle files are read on the timing path only, so convergence's output is unchanged.

**Commit:** 10: Report stops by settler in the timing report
