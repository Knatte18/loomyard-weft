# Card 10 — publish-retry

**Create:**
- `plan:internal/battenshed#ErrChildRunBusy` -> `var ErrChildRunBusy error`
- `plan:internal/battenshed#ChildPublishFailure` -> `type ChildPublishFailure struct`
- `plan:internal/battenshed#publishRetry` -> `type publishRetry struct`
- `plan:internal/battenshed#publishRetryCap` -> `const publishRetryCap = 30 * time.Minute`
- `plan:internal/battenshed#publishRetryFileSuffix` -> `const publishRetryFileSuffix = "-publish-retry"`
- `plan:internal/battenshed#recordMatchesHalt` -> `func recordMatchesHalt(record ChildPublishFailure, head string, status shedengine.Status, publishRow string) bool`
- `plan:internal/battenshed#innerRunProducer.publishRetryDecision` -> `func (p *innerRunProducer) publishRetryDecision(ctx context.Context, status shedengine.Status) publishRetry`
- `plan:internal/battenshed#innerRunProducer.publishRetryStep` -> `func (p *innerRunProducer) publishRetryStep(ctx context.Context, w *childWait) *waitEnd`
- `internal/battenshed/innerrun_publishretry_test.go`
- `plan:internal/battencli#readChildPublishFailure` -> `func readChildPublishFailure(taskLocation *lyxcwd.Location) (battenshed.ChildPublishFailure, bool, error)`
- `plan:internal/battencli#TestReadChildPublishFailure_MapsTheRecord` -> `func TestReadChildPublishFailure_MapsTheRecord(t *testing.T)`

**Edit:**
- `internal/battenshed#InnerRunDeps`
- `internal/battenshed#NewInnerRun`
- `internal/battenshed#innerRunProducer.Call`
- `internal/battenshed#childWait`
- `internal/battenshed#innerRunProducer.haltedStep`
- `internal/battenshed#innerRunProducer.noticeCondition`
- `plan:internal/battenshed#runMarkerSuffixes`
- `internal/battenshed/doc.go`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#childSpawnError`
- `internal/battencli#TestChildSpawnError`
- `internal/testkit/envkit#FullEnv`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shedengine#ShedBusyKind`
- `internal/verifytree#ReadPublishFailure`
- `internal/loomshed#NamePublish`
- `plan:internal/battenshed#ClearRunMarkers`

**Intent:**
Batten retries the first Publish verify failure of a batten run on its own (D5): it sends the child to the failure record's target producer and resumes it, and a second such failure wakes the orch.
This replaces battenshed's documented rule that batten never spawns or resumes a halted child, for this one case only.

New seams on `InnerRunDeps`:
`PublishRow string`, the child recipe's Publish row name, empty switching the retry off;
`PublishFailure func() (ChildPublishFailure, bool, error)`, the task worktree's Publish failure record, whose `ChildPublishFailure` holds `Head`, `Target` and `WrittenAt`;
`ChildHead func() (string, error)`, the task worktree's HEAD;
and `Goto func(ctx context.Context, target string) error`, the child's goto verb, whose refusal for a held run lock wraps the new `ErrChildRunBusy`.
`NewInnerRun` leaves the retry off when any of the three closures is nil.

The decision, `publishRetryDecision`, is made once per Call for a halted child, before the wait's first notice step, and kept on `childWait`; its process and git reads therefore run once per Call, never per check.
The marker `<scratchDir>/<row>-publish-retry` holds the retry's state and the time `started` was written: `started` before the goto, `goto-done` after a goto that succeeded, `resumed` after a resume that landed, `capped` when the cap's notice is queued.
`resumed` and `capped` each spend the run's one retry, and `publishRetryFileSuffix` joins `runMarkerSuffixes`, so only a fresh batten seed resets it.
- No marker, child blocked with `CurrentProducer` equal to `PublishRow`, and a record that `recordMatchesHalt` accepts: start the retry.
  `recordMatchesHalt` accepts a record with a non-empty `Target` and `WrittenAt` whose `Head` equals the child's HEAD and whose `WrittenAt` is strictly later than the history entry before the halting one and at or before the halting entry, the child's last, a Stuck of `PublishRow`; the three times compare as RFC 3339 whole-second UTC, and a halt with no earlier entry has no lower bound.
  Any other record, a stale one at the same HEAD from an earlier halt included, and a seam error, which is warned about, leave the halt to today's handling.
- `started` with the child still blocked at `PublishRow`: repeat the goto.
- `started` or `goto-done` with the child paused at the marker's target: resume it.
- Any other unfinished marker, such as `goto-done` with the child blocked at Publish again after a resume whose `resumed` write was lost, counts as the retry spent.

`publishRetryStep`, called from `haltedStep` before its usual work, carries the retry out:
the goto writes `started` first, then calls `Goto` with the record's target, logs at Info, writes `goto-done` and ends the wait with a budget-exempt Stuck naming the automatic retry and the target, which is its batten history entry;
the resume calls `Spawn`, as the awaiting arm does, so the child's own bootstrap resumes it, and on success writes `resumed`, logs at Info and ends the wait with a budget-exempt Stuck naming the resume.
A goto refused with `ErrChildRunBusy`, and a resume refused with `ErrChildNotParked` or `ErrChildDriverBusy`, are retried at most once per notice probe, the busy pane also getting the awaiting arm's driver-busy notice through `noticeDriverBusy`.
Measured from the marker's `started` time, so a restart does not reset it, the retrying stops at `publishRetryCap`: the marker becomes `capped` and the notice for the child's actual state goes, blocked at Publish for a refused goto or paused at the target for a refused resume, carrying the last refusal cut to one line.
A goto refused for any other reason, such as a target the recipe lacks, is not retried: the marker becomes `capped` and the blocked notice goes with the refusal.

Notices: `noticeCondition` returns no state-changed finding while the retry is to start or unfinished, that is for a child blocked at Publish the retry is about to take, and for a child paused at the target while the marker holds `started` or `goto-done`, however long the resume waits.
With the retry spent, `noticeCondition` sets the state-changed finding's `text` of a blocked-at-Publish child to `automatic Publish verify retry already ran`, which `conditionText` already renders after the halt text, so the orch knows the failure is the second; `conditionText` itself is unchanged.
Every other halt keeps today's handling: no spawn, no resume, one Warn per episode.

In `internal/battencli`, `battenCLI.wire` sets `PublishRow` to `loomshed.NamePublish`;
`PublishFailure` to `readChildPublishFailure` of the located task worktree, which reads the record with `verifytree.ReadPublishFailure` over the paths `readAgentActivity` already builds and maps its `Head`, `Target` and `WrittenAt`;
`ChildHead` to the task worktree fabric's `HeadSHA` (PATTERN-fabric-git);
and `Goto` to a run of `lyx loom goto --to <target>` in the task worktree's anchor, its output captured and its error passed through `childSpawnError`, logged at spawn and wait like `Spawn` (PATTERN-spawn-observability).
`childSpawnError` also wraps `battenshed.ErrChildRunBusy` for an output whose refusal kind is `shedengine.ShedBusyKind`.
`internal/testkit/envkit`'s `FullEnv` fills the three new closures and `PublishRow`.
`internal/battenshed/doc.go`'s halted-child paragraph and `docs/overview.md`'s batten entry state the automatic retry, its marker, its cap and that it is the one exception to batten never resuming a halted child.

Bound: once per batten run, a capped retry counting as that one; only for a child blocked at Publish with a record written during the halting attempt, which Publish writes only for a verify that exited non-zero; never for a record lacking the target or the write time; the goto and resume retries stop at `publishRetryCap`.

Tests, in `internal/battenshed/innerrun_publishretry_test.go`, over fake seams and a fake clock:
a matching record is sent to its target and resumed once, with no notice for the halt or the paused state, also when the resume waits past `noticeReportWait`, and a second Publish halt in the run gets the blocked notice saying the retry already ran;
a table of records that are not retried and get the blocked notice: none, one at another HEAD, a same-HEAD record from an earlier halt, one written in the same second as the entry before the halting attempt, ones lacking the target or the write time, and a `PublishFailure` or `ChildHead` read error, each warned about; and a record in the same second as the halting entry is retried;
a goto refused as busy, a resume refused as not parked and a resume refused with `ErrChildDriverBusy` are retried until they land, the driver-busy one also sending the awaiting arm's driver-busy notice once, and past the cap the notice for the child's actual state carries the last refusal, a restart with the marker present does not reset the cap, and a later Publish halt after the cap gets the blocked notice and no goto;
a goto refused for an unknown target is not retried and sends the blocked notice;
a batten restarted after the goto and before the resume resumes the child, and one restarted after `started` and before the goto repeats it.
`TestChildSpawnError` gains a row for a `shed_busy` refusal wrapping `ErrChildRunBusy`, and `TestReadChildPublishFailure_MapsTheRecord` writes a record in a task worktree and asserts the mapped fields, and found false for no record.
The new top-level tests raise the rows of `internal/battenshed` and `internal/battencli` in `cmd/lyx/testdata/test-budget.yaml`.

**ImpactSummary:** Run-Shed's halted arm gains one bounded automatic retry for a Publish verify failure and its notice suppression; every other arm and every other halt is unchanged.
