---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/battenshed/innerrun_publishretry_test.go:176
    summary: four top-level publish-retry tests never call t.Parallel although they touch no process-global state, against PATTERN-test-speed
  - id: F2
    severity: NIT
    class: design
    location: internal/battenshed/land.go:130
    summary: the pause Stuck is told apart from a failure Stuck by a string prefix of its reason, so the no-notice rule depends on wording
---

Range: the run has no `_lyx/loom/rework/` directory, so step 4 applies: `git diff $(git merge-base main HEAD)..HEAD` with `product.parent` = `main`.
The Publish failure section reads `none`.
I read the diff for all eleven cards against the plan and the focus directive, and ran `lyx gate test` over the touched packages and `lyx gate test --tags integration` over `battencli`, `gitrepo`, `battenshed`, `fabricengine`, `shedrecipe` and `battenrecipe`; all passed.

Focus questions settled, each with no defect found:

- Land producer and row (cards 8, 9): every failed step in `internal/battenshed/land.go` goes through `stop` or `failure` and ends Stuck naming the step, seam read errors included; only a cancelled context is a hard error.
  The pause path matches `land-row-pause`: the reason names `lyx batten goto <slug> --to Land` then `lyx batten run <slug>`, sends no notice, and `preGoto` in `internal/battencli/goto.go` admits that goto when batten's own row is Land.
- Run markers (cards 7, 8, 10): `runMarkerSuffixes` in `internal/battenshed/rownotice.go` names the row-notice, ready-snapshot and publish-retry suffixes, and `ClearRunMarkers` is called beside `ClearTerminalOpened` in both seed paths in `internal/battencli/arm.go`.
- Publish retry (cards 3, 10): `publishRetryDecision` starts only without a marker and only for a record `recordMatchesHalt` accepts, and every later marker state reads as spent; goto and resume retries stop at `publishRetryCap` from the marker's start time; the goto goes to the record's `Target`.
- Push retry (cards 1, 2): `retryablePushRefusal` accepts a remote `(failed)` and `cannot lock ref`, an exhausted retry wraps `ErrPushRetriesExhausted` and never `ErrPushRejected`, the exhausted check precedes `rebaseRetryTriggers`, and Add's branch pushes and the archive tag push use `gitrepo.PushWithRetry`.
- Notices (cards 6, 7): the done notice and the `final` flag are gone; `rowNotifier.failed` notifies once per history length and `send` makes at most `noticeMaxAttempts` tries spaced `rowNoticeRetrySpacing` apart.
- Test economy: no duplicate coverage found; the budget file rows were raised with the new tests.

### [BLOCKING] Publish-retry tests run serially without cause

**Location:** `internal/battenshed/innerrun_publishretry_test.go:176`, `:258`, `:313`, `:325`
**Issue:** `PATTERN-test-speed` requires `t.Parallel` unless a test touches process-global state.
`TestInnerRun_PublishRetrySendsTheChildToItsTargetAndResumesItOnce`, `TestInnerRun_PublishRetryRetriesRefusalsUntilTheyLandOrTheCapIsReached`, `TestInnerRun_PublishRetryDoesNotRetryAGotoRefusedForAnotherReason` and `TestInnerRun_PublishRetryContinuesFromTheMarkerAfterARestart` build a `retryHarness` over `t.TempDir()`, a fake clock and fake seams, and call neither `logcapture.Capture`, `t.Setenv` nor `t.Chdir`.
Only `TestInnerRun_PublishRetryDeclinesRecordsThatDoNotMatchTheHalt` needs to stay serial, since `logcapture.Capture` replaces the process-global logger output.
**Fix:** Call `t.Parallel()` first in the four tests, and in their subtests where the subtest uses the harness; leave the log-capturing test serial, with a header line saying why, as `notice_test.go` does.

### [NIT] Pause detected by reason prefix

**Location:** `internal/battenshed/land.go:130`
**Issue:** `stop` decides whether to queue the failure notice with `strings.HasPrefix(reason, pausedReasonPrefix)`, so the "a pause sends no notice" rule rests on the wording of a string `pausedReason` builds elsewhere.
A caller passing a reason that happens to start with that text, or a reworded `pausedReason`, silently changes notice behaviour.
**Fix:** Give `stop` an explicit `notify bool` parameter, or split a `pause` method from `stop`, instead of reading the rule back out of the reason text.
