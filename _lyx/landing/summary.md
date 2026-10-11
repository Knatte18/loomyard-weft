# Shuttle start: tolerate a lone %, probe sooner, name the cause

A shuttle start no longer fails on a ready Claude pane whose input line shows a stray `%`.
A start that never became ready now reports its cause and the pane's last lines.
The startup probe sees a ready provider within about a second instead of up to 10 s.
A seats chair's held-turn notice is deferred at most 10 minutes instead of 20.

Closes #549

## What changes

- **Lone `%` reads as an empty box.**
  In `internal/shuttleengine/claudeengine`, an input box whose only content after the caret and side bars is the single character `%` is blank.
  `IdleSession` reports idle and `InputBoxText` returns empty text for it.
  `%` beside other text, or any other single character, stays a draft.
  The `%` is most likely zsh's end-of-line mark, which Claude never received as input.
  The knowledge stays in the provider's classifier, per `PATTERN-shuttle-provider-seam`, and serves the pre-write guard, the startup sends and the orch watcher alike.
- **The start error reaches the producer.**
  `shuttleengine.Result` gains `StartError`, set only on `RunGated`'s not-started branch.
  `SingleLLMProducer.mapOutcome` appends it to its not-started message, so `errors.Is(err, shuttleengine.ErrNotStarted)` still holds.
- **A never-ready teardown ends with the pane's last lines.**
  When `abandonStartup` gets no cause, the provider never reached `StartupReady` and a startup capture exists, the error ends with that capture's last non-blank lines (`captureTail`, cut to `paneTailLines`).
  A teardown with a cause adds nothing, since send failures already carry a tail and the other causes report a mechanism failure with no screen.
  A pane that dies during the skill-load turn gets no tail.
- **Faster startup probe.**
  `awaitStartup` sleeps `poll_interval_ms` first and doubles the sleep while the pane capture does not change, up to the dismissal gap (`poll_interval_ms × liveness_every_n_polls`), resetting to the base when the capture changes.
  This follows the backoff rule in `PATTERN-daemon-wakeups`; the pattern file is not amended.
  The tick cap is computed from the base interval.
- **Startup-gate dismissals are throttled.**
  `checkLivenessTick` plays the trust-dismiss sequence at most once per dismissal gap on the run's clock.
  Any play that sent at least one key starts the gap, failed partway or not.
  The throttle only removes key presses, and the startup deadline and tick cap still end a gate that never clears.
- **`holdNoticeMaxDeferral` is 10 minutes.**
  Measured advisor turns ran from about 1.5 to 3.5 minutes.

`PATTERN-pane-text-injection.md`, `internal/shuttleengine/doc.go` and the `liveness_every_n_polls` comment in `template.yaml` are updated to match.
`cmd/lyx/testdata/test-budget.yaml` is raised for the new tests.

## Design decisions

- A ready provider still shows within the first one or two probes, because Claude's boot redraws the pane and resets the backoff.
- Spacing the dismissals rather than the probes keeps the protection the 10 s cadence gave against a second key landing in the next gate after a stale capture.
- Out of scope: opening the batten window before `loom start --no-attach` exits, Seed-Child's `main-weft` push timing, and `Run.Wait`'s own probe cadence.

## To check by hand

- Run `lyx gate test ./...` and `lyx gate test --tags integration ./...`; neither was run on this branch as a whole.
- Start a real shuttle run and confirm `lyx batten run <slug> --window` opens the window in about a second rather than about 11 s.
- If the `%` is in fact input in Claude's box, the new reading lets a send paste behind it, and the prompt reaches the agent with a leading `%`; no failing capture survived to confirm the end-of-line-mark reading.
- An operator draft of exactly `%` typed outside `operator_quiet_s` reads as empty, and the session idle probe's fallback for a strand with no run reads the box with no operator-activity hold.
- Another shell's end-of-line mark (`#` for a root zsh, or a custom `PROMPT_EOL_MARK`) still fails the same way.
- A gate needing more than one dismissal within one gap still takes a gap per extra key.
- A legitimately long advisor turn over 10 minutes now reaches the parent as a stall notice.
- Test files outside the changed behaviour's own packages were touched for helpers (`startup_test.go`, `fakes_test.go`, `hold_test.go`, `singlellm_test.go`, `claudeengine/start_test.go`, `claudeengine/session_test.go`); skim them for scope.
