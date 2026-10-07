---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/shuttleengine/claudeengine/background.go:4
    summary: "New and changed Go comments are hard-wrapped mid-sentence, breaking PATTERN-comment-line-breaks (background.go, hold.go, wait.go, rundir.go, attach.go, both doc.go files and several test files)"
  - id: F2
    severity: BLOCKING
    class: consistency
    location: internal/shuttleengine/hold_test.go:44
    summary: "New held-run tests omit t.Parallel and name no global state, breaking PATTERN-test-speed (TestWait_HeldTurnEndNotice, TestRun_Wait_HeldTurnEnd, TestDispositionCandidate_AskingReentry)"
  - id: F3
    severity: LOW
    class: scope
    location: internal/shuttleengine/spec.go:31
    summary: "Card 12 left prose that still names asking as a shuttle run outcome (spec.go, spec_test.go, run.go, claudeengine/settings.go, burlerengine/engine_test.go)"
  - id: F4
    severity: LOW
    class: consistency
    location: internal/shuttleengine/claudeengine/testdata/corpus/session-continued-in/events.jsonl:1
    summary: The session-continued-in corpus case replays the same Stop line to the same reading as issue411-prune-rest, so it adds no coverage
---

Range: `product.parent` is `main` and no `_lyx/loom/rework/` round exists, so the range is `git diff $(git merge-base main HEAD)..HEAD`, the sixteen card commits.

Evidence gathered: `go build ./...`, `go vet` under no tag and under `integration`, `tmux` and `llm`, and the untagged `go test ./...` all pass at HEAD.
`go test -tags integration` over `orchcli`, `shuttleengine` and `loomcli` passes.
`go vet ./...` passes at the commits of cards 5, 6, 9, 10 and 11, so the removal order builds commit by commit.
`grep` for `OutcomeAsking`, `AwaitOperator`, `ErrMasterAsking`, `MasterAskingError`, `runTriage`, `TriagePath` and `treadle-triage` finds nothing outside `_lyx/` and `docs/benchmarks/`.
`DeadReasonAsking` is untouched, as card 10 requires.

Focus questions settled with no defect:

- `outstandingBackgroundTasks` reads the transcript only when `background_tasks` is absent or not a list, and `notificationText` makes `queue-operation` and `queued_command` lines completions under the same after-the-tool-result rule (`TestParseEvents_BackgroundTasksKeyIsAuthoritativeOverTranscript` pins a notification before the result absorbing nothing).
- Every turn end without all output files is held on all three paths.
  `pollEventsTick` computes the offset once through `offsetPastEvent`, `recordWaiting` keeps it and `expiredTurnEnd` reports the same one.
  No hold touches `run.deadline`, and `finalize` no longer writes `AskingOffset`.
- `hold.go` follows the plan's notice shape: prefix, strand name, quoted `MessageTail`, at most five tasks, message start last.
  Every agent part goes through `delimitedAgentText`, which replaces control and delimiter characters and cuts to 200 runes.
  `NotifiedOffset` is persisted before the notifier call and restored when the write fails.
- `Attach` attaches a legacy `asking` record on a live tracked strand, resets it to `running`, and fills a missing `StrandName` from reed's status.
  `loomcli`, `webstercli` and `burlercli` hub runners call `SetNotifier` with `orchcli.NotifyPrime`; `shuttlecli`, `orchcli` and the detached runners do not.
- `asking` survives as the unexported `legacyAskingOutcome` in `attach.go`, apart from the prose in F3.
- The `Clock` rename leaves no use of the old type name, and `eventsSize` and `TestCollectAttachCandidates_RecordsEventsSize` are gone.

### [BLOCKING] Hard-wrapped new and changed comments

**Location:** `internal/shuttleengine/claudeengine/background.go:4-14`, `:74-75`, `:104-105`, `:193-196`; `internal/shuttleengine/claudeengine/doc.go:36-37`; `internal/shuttleengine/doc.go:69`; `internal/shuttleengine/hold.go:1-2`, `:24-25`, `:40-43`, `:70-72`; `internal/shuttleengine/rundir.go:109-112`; `internal/shuttleengine/wait.go:246-247`, `:560-561`, `:626-627`; `internal/shuttleengine/attach.go:284-285`, `:520-521`, `:539-540`; `internal/shuttleengine/run.go:191` (the `Result` doc); and test-file comments in `attach_asking_test.go:1-3` and `:54-55`, `wait_test.go:394-395`, `waiting_test.go:206-207`, `background_test.go:82-83`, `smoke_guardrail_test.go`, `smoke_interrupt_test.go` and `smoke_turns_test.go:230`.
**Issue:** `PATTERN-comment-line-breaks` requires one sentence per line with a further break only at an independent-clause boundary, and applies to new and changed comments.
The comments listed were written or reworded by this diff and break mid-sentence at a column, for example the `background.go` header's "...whose status is "running" are the whole / outstanding list ...".
Lines the diff did not touch stay as they are, since the pattern calls them legacy.
**Fix:** Reflow only the sentences this diff added or changed, one sentence per line, with no column limit and no change to wording.
Leave untouched legacy lines in the same paragraphs alone.

### [BLOCKING] New held-run tests run serially without naming global state

**Location:** `internal/shuttleengine/hold_test.go:44` (`TestWait_HeldTurnEndNotice`), `internal/shuttleengine/wait_test.go` (`TestRun_Wait_HeldTurnEnd`), `internal/shuttleengine/attach_asking_test.go:36` (`TestDispositionCandidate_AskingReentry`).
**Issue:** `PATTERN-test-speed` requires `t.Parallel` unless the test touches process-global state and a comment names it.
These tests build a per-test `newFixture` over `t.TempDir()`, fake reed and a fake clock, use no `logcapture`, env or cwd, and carry no comment.
The sibling tests in the same diff (`TestAttach_LegacyAskingAndHeldRuns` subtests, `messagetail_test.go`, `config_test.go`) are parallel, so the omission is not a convention of the file.
**Fix:** Add `t.Parallel()` at the top of each test and in its subtests, then run `go test -race -count=3` over `internal/shuttleengine` to confirm they are independent.
Add `t.Parallel()` to the top level of `TestAttach_LegacyAskingAndHeldRuns` as well.

### [LOW] Prose still names asking as a shuttle run outcome

**Location:** `internal/shuttleengine/spec.go:31`, `:146`, `:176`; `internal/shuttleengine/spec_test.go:26`; `internal/shuttleengine/run.go:480`; `internal/shuttleengine/claudeengine/settings.go:10`; `internal/burlerengine/engine_test.go:350`.
**Issue:** Card 12 requires that after it every remaining use of the word asking in Go comments is unrelated to a shuttle run outcome, and names the outcome lists to clear.
`burlerengine/engine_test.go` still lists "asking, died, timeout" as non-done outcomes.
The `spec.go` and `spec_test.go` comments speak of "an asking run" or "an asking outcome" being classified done.
`run.go:480` says Wait never reads a load turn end "as the run asking".
`settings.go:10` says the hook lets the run loop "classify it as a real-time asking signal", which Wait no longer does.
**Fix:** Reword each to the three outcomes and the hold: the stale-file comments speak of a run that stopped short of its output being classified done, the load-turn comment says Wait never reads one as a held turn end, the `settings.go` comment says the recorded live ask is held like any turn end without output, and the `burlerengine` comment lists died and timeout.

### [LOW] session-continued-in corpus case duplicates issue411-prune-rest

**Location:** `internal/shuttleengine/claudeengine/testdata/corpus/session-continued-in/` (`events.jsonl`, `case.yaml`, `transcript.jsonl`).
**Issue:** Card 15 requires each case to add a reading the synthetic rows and `skillload_test.go` do not.
The plan's reading for this case, Stop lines after the switch carrying the continuing session's transcript path, does not exist in the real run, and `case.yaml` correctly says so rather than synthesizing it.
What is left is the same single Stop line as `issue411-prune-rest/events.jsonl`, with `background_tasks: []`, the same `done` reading and a transcript the parser never reads because the payload list wins.
`PATTERN-test-economy` makes a replay that covers what `issue411-prune-rest` already covers a redundancy finding.
**Fix:** Delete the `session-continued-in` directory and report in the fixer report that the real run holds no turn end after the switch, so the plan's case cannot be built from real data.
