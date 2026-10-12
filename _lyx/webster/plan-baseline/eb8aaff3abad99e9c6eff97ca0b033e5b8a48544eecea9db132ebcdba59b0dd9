# Card 8 — batten-notice-log

**Create:**
- `internal/shedrun#NoticeLog`
- `internal/shedrun#NoticeLogRel`
- `internal/battenshed#NoticeLog`
- `internal/battenshed#appendNoticeLog`
- `internal/battenshed#TestAppendNoticeLog`
- `internal/battencli#noticeLog`
- `internal/battencli#TestNoticeLog_NamesTheHubLogAndTheQuotedAttach`

**Edit:**
- `internal/battenshed#RowNoticeDeps`
- `internal/battenshed#InnerRunDeps`
- `internal/battenshed#rowNotifier.send`
- `internal/battenshed#innerRunProducer.deliver`
- `internal/battenshed#noticeHarness`
- `internal/battenshed#noticeHarness.producer`
- `internal/battenshed#TestRowNotifierSend_DeliveryBound`
- `internal/battenshed#TestNotice_DeliveryIsRecordedOnlyWhenQueuedAndRetriedWithinItsCaps`
- `internal/battenshed/doc.go`
- `internal/battencli#rowNotices`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#Command`
- `internal/shedrun#TestPathConstructors`
- `internal/testkit/envkit#nilLegal`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Build D8: every notice a batten process composes is appended once, by that process, to one hub log file.

Path: `shedrun.NoticeLog(l)` is `<anchor>/.lyx/notices.log` (`lyxdirs.DotLyxDirName` joined with `notices.log`), and `NoticeLogRel()` its anchor-relative form; both are declared in `paths.go` with the other prime-scoped paths.
The file sits directly under the prime's `.lyx`, outside `shedrun.RunsRoot`, `shedrun.ScratchRoot`, `.scratch` and the lyx temp root, which are every directory `lyx cleanup` sweeps (`battencli`'s cleanup walks the two shed roots only), so no cleanup segment removes it or reads it as a run.
The branch already contains `lyx cleanup` (bc9ee7a02).

`battenshed.NoticeLog` carries `Path` (empty switches the log off), `AttachCommand` (the run's attach line) and `Now func() time.Time` (nil reads `time.Now`).
`appendNoticeLog` opens `Path` with `O_APPEND|O_CREATE|O_WRONLY` and writes one line, `<UTC time, RFC 3339>\t<notice text>\t<attach command>\n`, in one `Write`, so concurrent batten processes never interleave inside a line; a failure to open or write is a `logger.Warn` naming the path and never blocks or changes the notice.
It uses the standard library and `logger` only, so `battenshed`'s import allowlist is unchanged.
`RowNoticeDeps` and `InnerRunDeps` each gain a `NoticeLog NoticeLog` field.
`envkit`'s `nilLegal` gains the three paths `InnerRun.NoticeLog.Now`, `Land.Notices.NoticeLog.Now` and `Teardown.Notices.NoticeLog.Now`, each nil-legal because a nil `Now` reads `time.Now`, the same reason `InnerRun.Now` and `Land.Notices.Sleep` are listed there; `wire` leaves them nil, so without them `envkit.NilSeams` reports them and the seam-completeness tests fail.

Once per composition, at the first delivery attempt in this process and never at a later one:
- `rowNotifier.send` appends on entry, before the orch-strand check, so a row notice is logged also when it is held back for a missing orch strand or dropped after its attempts; a held-back row notice is composed again by the next failure at the same history length and logged again then.
- `innerRunProducer.deliver` appends where it creates the episode's `noticeDelivery` for a key, so the later retries of that key in this process log nothing more, while a restarted batten (a fresh producer) composing the same episode's notice again logs it again.
The append is keyed on nothing persisted, as D8 states.

`battencli.noticeLog(location, slug)` resolves the path through `shedrun.NoticeLog(location)` and the attach command as `shell.ForGOOS().ChangeDir(<pair dir>) + " && lyx reed attach"`, the pair dir `fabricengine.WorktreePath(location, slug)` quoted by the shell seam (`PATTERN-shell-mechanics-seam`); `rowNotices` (Land and Worktree-Teardown) and `wire`'s `InnerRun` set it (`PATTERN-told-geometry`).
`Command`'s `Long` gains a paragraph naming the log's path, composed from `shedrun.NoticeLogRel()` rather than spelling the directory (`PATTERN-lyxdirs-single-declarer`), and what one line holds.
`battenshed/doc.go`'s notice paragraph states the log.

Tests:
- `TestAppendNoticeLog` (new, `rownotice_test.go`): a line carries the fixed clock's UTC time, the text and the attach command, tab-separated, and ends in one newline; two goroutines appending at once leave two whole lines; a path under a regular file (unwritable) logs a warning, writes nothing and returns; an empty path writes nothing.
- `TestRowNotifierSend_DeliveryBound`: every row passes a temp `NoticeLog` and wants exactly one log line per `send`, the no-strand row and the never-queued row included; a new row with an unwritable log path (under a regular file) still makes its Notify call, is queued and returns true.
- `TestNoticeLog_NamesTheHubLogAndTheQuotedAttach` (new, `wire_test.go`): `noticeLog` over a `locationkit` prime whose hub path holds a space returns `shedrun.NoticeLog` of the prime as `Path` and `cd '<hub>/<slug>' && lyx reed attach` as `AttachCommand`, the directory quoted.
- `noticeHarness` gains a `noticeLog` path and `noticeHarness.producer` sets `NoticeLog` from it with the harness clock; `TestNotice_DeliveryIsRecordedOnlyWhenQueuedAndRetriedWithinItsCaps` wants one log line in the queued sub-test, one in each three-attempt retry sub-test, one in the no-strand sub-test across both of its runs over one producer, and two in the restarted-batten sub-test, one per producer.
- `TestPathConstructors` gains a `NoticeLog` row.
- `test-budget.yaml`: `internal/battenshed` 104 → 105, `internal/battencli` 68 → 69.

**ImpactSummary:** two dep structs gain one field each, and nil-equivalent zero values keep every existing producer test passing; battencli wiring and help text; one new shedrun path; envkit's nil-legal list gains the three `NoticeLog.Now` paths.
