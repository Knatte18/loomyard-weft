# Card 8 — drop-hub-notice-log

**Edit:**
- `internal/battenshed#RowNoticeDeps`
- `internal/battenshed#InnerRunDeps`
- `internal/battenshed#rowNotifier.send`
- `internal/battenshed#innerRunProducer.deliver`
- `internal/battenshed#HandBackNotice`
- `internal/battenshed#TestRowNotifierSend_DeliveryBound`
- `internal/battenshed#noticeHarness`
- `internal/battenshed#newNoticeHarness`
- `internal/battenshed#noticeHarness.producer`
- `internal/battenshed#TestNotice_DeliveryIsRecordedOnlyWhenQueuedAndRetriedWithinItsCaps`
- `internal/battenshed#TestHandBackNotice`
- `internal/battencli#rowNotices`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#Command`
- `internal/shedrun#TestPathConstructors`
- `internal/testkit/envkit#nilLegal`
- `internal/battenshed/doc.go`
- `PATTERN.md`
- `pattern/PATTERN-durable-vs-ephemeral-state.md`

**Delete:**
- `internal/battenshed#NoticeLog`
- `internal/battenshed#appendNoticeLog`
- `internal/battenshed#TestAppendNoticeLog`
- `internal/battenshed#noticeLogClock`
- `internal/battenshed#fixedNoticeLog`
- `internal/battenshed#readNoticeLogLines`
- `internal/battenshed#unwritableNoticeLogPath`
- `internal/battencli#noticeLog`
- `internal/battencli#noticeLogHelp`
- `internal/battencli#TestNoticeLog_NamesTheHubLogAndTheQuotedAttach`
- `internal/shedrun#NoticeLog`
- `internal/shedrun#NoticeLogRel`
- `internal/shedrun#noticeLogFileName`

**Intent:**
The hub notice log goes (decision record D8): a creel notice stays unread until the orch reads it, and the journal holds it after, so the log is a second copy.

Batten stops appending to `.lyx/notices.log`: `NoticeLog` and `appendNoticeLog` go, `RowNoticeDeps` and `InnerRunDeps` lose their `NoticeLog` field, `rowNotifier.send` and `innerRunProducer.deliver` lose their append, and `HandBackNotice`'s comment drops the log.
`battencli` loses `noticeLog`, the `NoticeLog` values in `rowNotices` and `battenCLI.wire`, and `noticeLogHelp`'s paragraph in `Command`'s help.
`shedrun` loses `NoticeLog`, `NoticeLogRel` and `noticeLogFileName`, and `envkit`'s `nilLegal` loses its three `NoticeLog.Now` entries.
Delivery, markers, attempts and control flow of every notice are unchanged.
The orch role's fallback already reads creel (card 6), and `lyx orch cleanup --apply` removes the retired file (card 9).

PATTERN changes, as the decision record lists them:
`PATTERN-durable-vs-ephemeral-state`'s entry line in `PATTERN.md` becomes "Adding a file: tracked under `_lyx`, never-tracked under `.lyx`, same subpath; nothing a run, the orch or a landing depends on lives only under `.lyx`." with its background link kept, and `pattern/PATTERN-durable-vs-ephemeral-state.md` gains, verbatim, the sentence: "Nothing a run, the orch or a landing depends on lives only under `.lyx`: decisions, landings, stop reports and board state are always in git (board, records, decision record), and never-tracked state such as the creel journal is a delivery path or cache that a fresh machine starts without."

Tests:
`TestAppendNoticeLog` and its helpers go with the function.
`TestRowNotifierSend_DeliveryBound`, `TestHandBackNotice`, the notice harness (`noticeHarness`, `newNoticeHarness`, `noticeHarness.producer`) and `TestNotice_DeliveryIsRecordedOnlyWhenQueuedAndRetriedWithinItsCaps` drop their notice-log fields and line-count assertions and keep every delivery assertion.
`TestNoticeLog_NamesTheHubLogAndTheQuotedAttach` goes; `TestPathConstructors` drops its `NoticeLog` and `NoticeLogRel` rows.
`internal/battenshed/doc.go` drops the "Notice log" paragraph.

**ImpactSummary:** Removes one write-only file seam from batten and its two path helpers from shedrun; no reader of the log remains, and notice delivery is untouched.
