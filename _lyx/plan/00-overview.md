---
format: 5
approved: true
language: go
---

# Plan: shuttle-send — sends that land, and stops that are recorded

A text shuttle sends into an agent pane must land, and when it cannot, the run retries instead of halting (#442, #443, and the "no send into a running turn after a restart" case).
Every verified send first waits for an idle session, settles its typed text before Enter, confirms submission within a window and clears its own text when it did not land; a start-time send that does not land tears the run down as not started, and a gate re-prompt that does not land is re-sent on a later tick.
A shuttle run lyx itself stopped is recorded `stopped` through one shuttle stop verb, which burler, webster and attach use in place of their own strand removals, so a later attach probe never halts on it (#444).

## Card Index

1 — shuttle-send-config — add `send_ready_timeout_s` and `submit_confirm_timeout_s` and reword the two settle keys
2 — claude-type-and-clear — provider seam gains typing without submit, a clear choreography and a paste-placeholder reading
3 — send-waits-for-idle — every verified send waits for an idle session or fails `ErrSessionBusy` with the pane's last lines
4 — send-settles-and-confirms — settle before Enter, confirm within a window, fail `ErrSubmissionNotLanded` and clear own text
5 — start-send-teardown — a start-time send or persist that fails tears the run down as not started
6 — gate-boundary-idle — gate sends wait for an idle writer, and an unlanded re-prompt retries without spending an attempt
7 — shuttle-stop-verb — shuttle's stop verb records `stopped` before removing a run's strand; attach uses it
8 — burler-stops-through-handle — burler stops a half through its run handle's `Stop`
9 — webster-stops-through-shuttle — webster's reclaim stops through `Runner.StopStrand`; `PATTERN-shuttle-stop` lands
10 — delete-strand-removers — delete burler's, shedfake's and webster's removers

## Shared Decisions

### Decision: send-context

Every verified send takes one `sendContext` (reed, engine, strand guid, events path, `Config`, `Clock`, deadline).
The send's timed waits (the idle polls, the appearance polls, the settle reads and the confirm backoff) sleep and read time through that `Clock`, so tests drive them with the existing `fakeClock`; `playInputs`' per-step `SettleMS` stays on `inputSleep`.
A zero deadline means none; only the wait loop's `sendWithin` sets one.

### Decision: idle-reading-capability

"The idle reading" is `SessionCycler` (`Startup` ready and `IdleSession`).
An engine without it keeps today's `requireReadyAgentPane` guard alone and reads no session signals, since D1's 10 s override is timed by the idle reading; it gets no clear either.

### Decision: settle-and-backoff-sources

The settle-read interval is shuttle's own `Config.SubmitSettleMS`; the first confirm interval is `InputBoxReader.SubmitSettle()`, which Claude builds from the same config's `submit_redraw_settle_ms`.
A non-positive `SendReadyTimeoutS` or `SubmitConfirmTimeoutS` in a hand-built `Config` is floored to the template default, as `pollInterval` floors its key; `LoadConfig` refuses both.

### Decision: clear-capability

The clear choreography and the paste-placeholder reading are one new optional capability, `InputBoxClearer`, so `shuttleengine` holds no Claude TUI shape (`PATTERN-shuttle-provider-seam`).

### Decision: stop-verb-surface

The handle form is `Run.Stop() error`, which `burlerengine.Handle` gains; the guid form is `Runner.StopStrand(guid string) error`, which `websterengine.StrandStopper` names.
Attach's superseded removal uses the package-internal `stopRecorded` over the candidate's run dir.

## verify:

```
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
