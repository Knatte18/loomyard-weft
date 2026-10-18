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

Overview headroom:
`patternlint.Lint` refuses a branch that grows `PATTERN.md` past `max(base, pattern.HeadroomBytes)`, measured from the merge base at every verify gate, and the base sits 4 bytes under `HeadroomBytes`.
This entry rewrite adds 73 bytes and card 10's `PATTERN-orch-pane-single-writer` rewrite adds 84, so this card, the first to grow the overview, makes room for both by shortening these entry lines, each keeping its name, its `(test)` mark and its background link, if any, as they stand:

- `PATTERN-told-geometry`: "Writing an engine: it is handed its absolute paths and derives none, never importing `internal/lyxcwd`."
- `PATTERN-mutation-record`: "Adding a mutating fabric verb: it fills a `*Mutations` record and its result embeds `MutationRecord`."
- `PATTERN-shed-recipe-registry`: "Registering a shed producer: one `map[string]Constructor` in `internal/shedrecipe` behind `Lookup`/`Names`; no `init()`, `Register` or `lyxcwd`."
- `PATTERN-creel-store`: "Writing the creel journal or reaching a pane: only `internal/creelengine`, a pane only via its `Delivery` seam."
- `PATTERN-completion-signal`: "Finding a run unfinished in `internal/shuttleengine`: consult `allOutputFilesExist` first."
- `PATTERN-spawn-observability`: "Starting an OS process from `lyx`: log the spawn, and a teardown it waits for, via `internal/logger`."
- `PATTERN-documentation-lifecycle`: "Design docs: none for unbuilt work; a built design lives in its package's `doc.go`."

Together they cut 159 bytes, so `PATTERN.md` ends this card 86 bytes under its base size and card 10's rewrite leaves it 2 under (15558 against a base of 15560).
`PATTERN-plan-generation` keeps its line, since its "top-level files" qualifier is what tells the `archive-*/` subdirectories of `_lyx/plan/` apart from the one generation.
Each background file already states the detail its line drops (the envelope keys, the real process and the negative answers among them), so no `pattern/` file changes for the cuts.

Tests:
`TestAppendNoticeLog` and its helpers go with the function.
`TestRowNotifierSend_DeliveryBound`, `TestHandBackNotice`, the notice harness (`noticeHarness`, `newNoticeHarness`, `noticeHarness.producer`) and `TestNotice_DeliveryIsRecordedOnlyWhenQueuedAndRetriedWithinItsCaps` drop their notice-log fields and line-count assertions and keep every delivery assertion.
`TestNoticeLog_NamesTheHubLogAndTheQuotedAttach` goes; `TestPathConstructors` drops its `NoticeLog` and `NoticeLogRel` rows.
`internal/battenshed/doc.go` drops the "Notice log" paragraph.

**ImpactSummary:** Removes one write-only file seam from batten and its two path helpers from shedrun; no reader of the log remains, and notice delivery is untouched; the seven shortened `PATTERN.md` entry lines keep their meaning, each background file holding the detail its line drops, and `TestOverview_RealFilePassesCheck` checks their format.
