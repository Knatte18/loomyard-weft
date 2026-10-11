# Card 7 — row-failure-notice

**Create:**
- `plan:internal/battenshed#RowNoticeDeps` -> `type RowNoticeDeps struct`
- `plan:internal/battenshed#rowNotifier` -> `type rowNotifier struct`
- `plan:internal/battenshed#newRowNotifier` -> `func newRowNotifier(producer, slug, scratchDir string, deps RowNoticeDeps) rowNotifier`
- `plan:internal/battenshed#rowNotifier.failed` -> `func (n rowNotifier) failed(ctx context.Context, step, reason string)`
- `plan:internal/battenshed#rowNotifier.send` -> `func (n rowNotifier) send(ctx context.Context, line string) bool`
- `plan:internal/battenshed#rowNoticeFileSuffix` -> `const rowNoticeFileSuffix = "-row-notice"`
- `plan:internal/battenshed#rowNoticeRetrySpacing` -> `const rowNoticeRetrySpacing = 30 * time.Second`
- `plan:internal/battenshed#runMarkerSuffixes` -> `var runMarkerSuffixes []string`
- `plan:internal/battenshed#ClearRunMarkers` -> `func ClearRunMarkers(scratchDir string) error`
- `plan:internal/battenshed#TestWorktreeTeardown_FailureNoticeOncePerEpisode` -> `func TestWorktreeTeardown_FailureNoticeOncePerEpisode(t *testing.T)`
- `internal/battenshed/rownotice_test.go`
- `plan:internal/battencli#battenHistoryLen` -> `func battenHistoryLen(location *lyxcwd.Location, slug string) (int, error)`
- `plan:internal/battencli#rowNotices` -> `func rowNotices(location *lyxcwd.Location, slug string) battenshed.RowNoticeDeps`
- `plan:internal/battencli#TestWire_BattenStatusSeamsReadBattensOwnStatus` -> `func TestWire_BattenStatusSeamsReadBattensOwnStatus(t *testing.T)`

**Delete:**
- `internal/battencli#TestWire_PauseRequestedReadsBattensOwnStatus`

**Edit:**
- `internal/battenshed#TeardownDeps`
- `internal/battenshed#worktreeTeardownProducer`
- `internal/battenshed#NewWorktreeTeardown`
- `internal/battenshed#worktreeTeardownProducer.Call`
- `internal/battenshed/doc.go`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#battenCLI.battenPreRun`
- `internal/battencli#battenCLI.battenPreStepUnmarked`
- `internal/battencli#TestBattenPreStep_SeedsAbsentStatus`
- `internal/testkit/envkit#FullEnv`
- `internal/testkit/envkit#nilLegal`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
The rows after Run-Shed get a failure notice (D4), built once here and given to Worktree-Teardown; card 8's Land row reuses it.
The new code lives in a new file `internal/battenshed/rownotice.go`.

`RowNoticeDeps` carries the row's notice seams:
`Notify` (the same `func(ctx, line) (queued bool, err error)` shape as `InnerRunDeps.Notify`; nil switches the notices off),
`OrchStrandRecorded` (nil reads as recorded),
`BattenHistoryLen func() (int, error)`, the length of batten's own status history,
and `Sleep` (nil is `waitOrCancel`), the pause between delivery attempts.

`rowNotifier.failed` queues one notice per failure episode, `[batten notice] <slug>: <row> failed at <step>: <reason cut to one line by oneLine and noticeErrorMax>; way forward: fix the cause, then run "lyx batten run <slug>" from the prime`.
An episode is one entry of the row from its predecessor's Done or from a halt at that row, told apart by batten's history length:
the marker `<scratchDir>/<row>-row-notice` keeps a `failure <length>` line, and a failure whose current history length equals the recorded one is the same episode and sends nothing.
A halt appends the row's Stuck or failed entry, so a `lyx batten run <slug>` after it that fails again sends a new notice, while a batten that died inside the row appended nothing and so never repeats a queued notice.
A `BattenHistoryLen` error is warned about and reads as a new episode, since a repeated notice costs less than a lost one.
`rowNotifier.send` makes at most `noticeMaxAttempts` attempts spaced by `rowNoticeRetrySpacing` through `Sleep`, stopping early on cancellation;
with no orch strand recorded it does not call Notify and warns once;
a notice still unqueued after the attempts is dropped with a Warn, which leaves the failed row halted and visible in `lyx batten status`.
The marker line is written only once the notice was queued, and a marker write failure is a Warn.
A notice is informational: it never changes the row's outcome, and the attempts delay the halt by at most the spacing times the attempts.

`ClearRunMarkers` removes every file in `scratchDir` whose name ends in a suffix of `runMarkerSuffixes`, which holds `rowNoticeFileSuffix` (cards 8 and 10 add theirs); an absent file or directory is not an error.
`battenCLI.battenPreRun` and `battenCLI.battenPreStepUnmarked` call it beside `ClearTerminalOpened` when they seed a fresh batten status, so a re-run of a slug never inherits an earlier run's markers.

`TeardownDeps` gains `Notices RowNoticeDeps`; `NewWorktreeTeardown` keeps its signature and builds the producer's notifier from `deps.Notices`.
`worktreeTeardownProducer.Call` calls `failed` on every Stuck it returns and every hard error it returns under a live context, naming the step: `prime lock`, `session shutdown` or `worktree removal`, so a Worktree-Teardown failure sends one notice.
A cancelled context sends none.

In `internal/battencli`, `battenHistoryLen` reads batten's own status file the way the Run-Shed `PauseRequested` seam does and returns its history length, 0 for an absent file;
`rowNotices` returns `RowNoticeDeps` whose `Notify` and `OrchStrandRecorded` are the closures Run-Shed's notices use (`orchengine.QueueNotice` on the prime's orch, `orchStrandRecorded`) and whose `BattenHistoryLen` is `battenHistoryLen`;
`battenCLI.wire` sets `Teardown.Notices` to it.
In `internal/testkit/envkit`, `FullEnv` fills `Teardown.Notices.OrchStrandRecorded` and `Teardown.Notices.BattenHistoryLen`, and `nilLegal` lists `Teardown.Notices.Notify` and `Teardown.Notices.Sleep`, following `InnerRun.Notify` and `InnerRun.Sleep`.
`internal/battenshed/doc.go` gains a paragraph on the row notices: who sends them, the episode rule, the delivery bound and the marker reset at a fresh seed.

Tests:
`TestWorktreeTeardown_FailureNoticeOncePerEpisode` drives a failing Shutdown three times with a fake Notify and history length: one notice naming `session shutdown` and the way forward, none on a re-entry at the same history length, a new one after the length grows, and a new one, with a Warn, when `BattenHistoryLen` errors after a notice was queued; and none for a cancelled context;
`internal/battenshed/rownotice_test.go` covers `rowNotifier.send`'s missing strand, retry spacing and drop after `noticeMaxAttempts`, and `ClearRunMarkers` removing a marker for every suffix in `runMarkerSuffixes` while leaving another file;
`TestWire_PauseRequestedReadsBattensOwnStatus` becomes `TestWire_BattenStatusSeamsReadBattensOwnStatus`, one table over the wired seams that read batten's own status file on the same fixture: its `InnerRun.PauseRequested` rows (absent, flag clear, flag set) stay, and `Teardown.Notices.BattenHistoryLen` rows assert a status with a known history answers its length and an absent status answers 0;
the old test is deleted rather than extended because its name would no longer describe its rows;
`TestBattenPreStep_SeedsAbsentStatus` also asserts a pre-existing row-notice marker is gone after the fresh seed.
The new top-level tests raise `internal/battenshed`'s row in `cmd/lyx/testdata/test-budget.yaml`; `internal/battencli`'s count is unchanged, one test deleted and one created.

**ImpactSummary:** Worktree-Teardown gains an informational failure notice and batten's fresh seed clears one more marker; the teardown's verdicts and its constructor's signature are unchanged.
