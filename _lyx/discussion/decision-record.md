# Decision record — shuttle-startup-ready

## Goal

Shuttle starts stop failing on a ready Claude pane whose input line shows a stray `%`, a not-started producer failure names its cause and the pane's last lines, every shuttle start passes its startup probe within about a second of the provider becoming ready, and a seats chair's held-turn notice is deferred no longer than 10 minutes.

## Scope

In:

- Claude's input-box reading treats a box holding only a lone `%` as empty (issue #549).
- A not-started run's start error reaches the single-LLM producer's failure message, with the pane's last lines.
- `awaitStartup` probes every `poll_interval_ms`, and a startup-gate dismissal is replayed at most once per `poll_interval_ms × liveness_every_n_polls`.
- `holdNoticeMaxDeferral` drops from 20 to 10 minutes.

Out:

- Opening the batten window before `loom start --no-attach` exits (board Part 2, idea 2).
- Seed-Child's `main-weft` push timing and segment-color typing before the window opens (board Part 2, idea 3).
- Extra retries or longer idle waits for the startup sends.
- Any change to Wait's own probe cadence for a run that is already past startup or is re-attached.
- Any change to the seats `advisorWork.outstanding` predicate.

## Decisions

### D1 — Claude reads a lone `%` in the input box as an empty box

Decision: in `internal/shuttleengine/claudeengine`, the input-box interior reading counts a box whose only non-blank content, after the caret and side bars are stripped, is the single character `%` as blank.
`IdleSession` then reports idle for it and `InputBoxText` returns empty text for it.
Any other content, `%` beside other text included, stays a draft.
Bound: the one character `%` alone in the box; nothing else is let through, and the operator-activity hold still runs on every reading.
A real operator draft of exactly `%` typed outside reed's quiet window would read as empty and a paste would merge with it; that is the whole of what the change lets through.
Rationale: the `%` is most likely zsh's `PROMPT_EOL_MARK`, which the pane's login shell prints before reed's launch line starts Claude and which a pane resize can leave on the row Claude later draws its box on; Claude would then never have received it as input, so the box is empty in fact and only its rendering carries the mark.
Nothing in a shuttle pane types a lone `%` into Claude's box, and a later agent on the same Claude Code version in the same worktree started cleanly.
The fix belongs in the provider's classifier, per `PATTERN-shuttle-provider-seam`, and it serves every caller of the reading: shuttle's pre-write guard, the startup sends and the orch watcher alike.

### D2 — The pane-text-injection invariant names the `%` reading

Decision: `pattern/PATTERN-pane-text-injection.md` gains one line under the pre-write guard bullet: the provider's box reading defines "empty", and Claude's reads a lone `%`, the shell's end-of-line mark left on screen, as empty.
Rationale: D1 narrows what "the input box is empty" admits; a narrowed guard is stated in the invariant's background, not left for a reader to find in the classifier.

### D3 — A not-started run carries its start error on `Result`

Decision: `shuttleengine.Result` gains a `StartError string`, set only on `RunGated`'s not-started branch to the text of the error `start` returned, and empty on every other return.
`SingleLLMProducer.mapOutcome` appends it to its not-started failure message after the existing `shuttle run outcome <outcome>: <ErrNotStarted>` text, so `errors.Is(err, shuttleengine.ErrNotStarted)` still holds.
Rationale: `RunGated` folds the not-started error into `(result, nil)`, which discards the cause; the halted step then shows only "the provider never became ready" while the trace held the real cause.
The multi-LLM chair path already returns the seats engine's error with its full text through `errorExit`, and burler already carries `StartError`, so only the single-LLM path needs it.

### D4 — A never-ready teardown's error ends with the pane's last lines

Decision: when `abandonStartup` runs with no cause (the provider never reached `StartupReady`) and a startup capture exists, its returned error ends with the last non-blank lines of that capture, cut to the same line count a send failure uses (`paneTailLines`).
When a cause is present, nothing is added, since a busy-send cause already carries the pane's last lines.
Rationale: with D3 the producer message then always shows the screen that ended the start, without opening the trace or the saved capture file.

### D5 — `awaitStartup` probes every `poll_interval_ms`

Decision: `awaitStartup` sleeps `pollInterval(cfg)` between probes instead of `pollInterval(cfg) × liveness_every_n_polls`, and its tick cap is computed from that interval.
Each probe still runs `checkLivenessTick`: one reed status and, before ready, one pane capture.
Rationale: the provider is ready within about a second, and the 10-second sleep is what makes every shuttle start, and so every batten window, take about 11 seconds.
The loop is bounded by `startup_timeout_s` and its tick cap, so `PATTERN-daemon-wakeups` does not apply, and one probe a second is at its floor anyway.

### D6 — A startup-gate dismissal is replayed at most once per dismissal gap

Decision: `checkLivenessTick` plays the engine's trust-dismiss sequence only when no dismissal was played on this run within the dismissal gap, `pollInterval(cfg) × liveness_every_n_polls` (10 s under the shipped template), read on the run's clock.
A probe inside the gap that still sees a gate plays nothing and classifies the startup window as before.
The first gate a run sees is dismissed on the probe that sees it.
Bound: the throttle only removes key presses; it never adds one, and the startup deadline and tick cap still end a gate that never clears.
Rationale: the 10-second cadence existed so a capture taken before Claude redraws after an Enter cannot drive a second key into the next gate; spacing the dismissals rather than the probes keeps that protection while D5 makes readiness visible within a second.
Wait's own startup probe already runs at the dismissal gap, so the throttle never holds a dismissal there.
`template.yaml`'s comment on `liveness_every_n_polls` says it also spaces startup-gate dismissals; no config key is added.

### D7 — `holdNoticeMaxDeferral` is 10 minutes

Decision: `holdNoticeMaxDeferral` in `internal/shuttleengine/hold.go` drops from 20 to 10 minutes; `holdNoticeSettle` and the predicate are unchanged.
Rationale: advisor turns measured in this hub's transcripts ran from about 1.5 to 3.5 minutes, so 10 minutes leaves about three times the longest observed batch while halving how long a wedged advisor can hide a chair stall.
An advisor that is held or dead already stops deferring: `advisorWork.outstanding` skips an ended run, and a held advisor's session reading is no longer busy with cause turn.
So the only case the cap governs is an advisor stuck mid-turn, which no reading can tell from a long batch.

## Constraints

- `PATTERN-shuttle-provider-seam`: the `%` knowledge lives only in `claudeengine`; `shuttleengine` gains no Claude specifics.
- `PATTERN-pane-text-injection`: D1 narrows "empty box"; D2 records it in the background file in the same commit.
- `PATTERN-completion-signal`: every negative startup answer still goes through `checkLivenessTick`, `classifyStartupWindow`, `classifyDeadlineExpiry` or a direct `allOutputFilesExist`; D5 and D6 change cadence and key replay only.
- `PATTERN-daemon-wakeups`: `awaitStartup` is a bounded wait; its one-second sleep is at the floor, and no sub-second literal is added.
- `PATTERN-test-speed`: cadence and throttle tests inject the run clock and never wait out an interval.
- `PATTERN-test-economy`: each new scenario extends an existing table or test in its package where one covers the surface.
- `PATTERN-comment-line-breaks`: the rewritten `awaitStartup` probe-cadence comment and the new comments keep one sentence per line.
- `PATTERN-documentation-lifecycle`: `internal/shuttleengine/doc.go` is updated where it states the startup cadence or the deferral cap.
- `PATTERN-refusal-way-forward` does not apply: no new refusal is added.

## Auto-mode assumptions

- The `%` is zsh's end-of-line mark left on screen, not input Claude received; no startup capture of the failing run survived to confirm it.
- Advisor batch times from 9 measured turns stand for typical advisor work.
- The board's Part 2 title, "probe every second", scopes Part 2 to idea 1.
- An operator does not type a lone `%` as a draft into a shuttle pane.

## Open risks

- If the `%` is in fact input in Claude's box, D1 lets a send paste behind it, and the prompt reaches the agent with a leading `%`.
- Another shell's end-of-line mark (`#` for a root zsh, or a custom `PROMPT_EOL_MARK`) still fails the same way.
- A gate that needs more than one dismissal within 10 seconds still takes 10 seconds per extra key, as today.
- A legitimately long advisor turn over 10 minutes now reaches the parent as a stall notice.

## Acceptance criteria

- A capture with a ready Claude box holding only `%` after the caret classifies idle, and its box text reads empty.
- A box holding `%` beside other text, or any other single character, still classifies not idle and reads that text.
- A start whose pane shows a ready box with a lone `%` passes the color send and the skill-load send and returns a run.
- `awaitStartup` reaches ready on the first probe after the pane shows ready, under an injected clock, with sleeps of one poll interval.
- A pane that shows a gate on consecutive one-second probes gets one dismissal per dismissal gap, not one per probe; a gate that never clears still ends the start at the startup deadline as died.
- A single-LLM producer whose shuttle start never became ready fails with a message that still wraps `ErrNotStarted` and contains the start error's text.
- A never-ready start's error ends with the last lines of its last startup capture.
- `Result.StartError` is empty on every return other than `RunGated`'s not-started branch.
- A chair held while an advisor reads mid-turn notifies at the first tick at or past 10 minutes.
- `lyx gate test ./...` and `lyx gate test --tags integration ./...` pass.

## Notes for the plan writer

- `internal/shuttleengine/wait.go`: `awaitStartup`, `startupTickCap`, `checkLivenessTick`, `abandonStartup`.
- `internal/shuttleengine/send.go`: `paneTail` and `paneTailLines` already cut a pane's last lines for send failures.
- `internal/shuttleengine/run.go`: `Result`, `RunGated`.
- `internal/shuttleengine/claudeengine/session.go`: `inputBoxInterior`, `isBlankBoxInterior`, `IdleSession`, `InputBoxText`.
- `internal/shedadapters/singlellm.go`: `mapOutcome`.
- `internal/shuttleengine/hold.go` and `hold_test.go`, which reads `holdNoticeMaxDeferral`.
- `cmd/lyx/testdata/test-budget.yaml` if a new top-level test pushes a package past its row.
