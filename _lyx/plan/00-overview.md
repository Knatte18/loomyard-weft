---
format: 5
approved: false
language: go
---

# Plan: shuttle startup is ready-fast and says why it failed

Shuttle starts fail on a ready Claude pane whose input box shows a stray `%`, the shell's end-of-line mark, because the box reading counts it as a draft.
A not-started single-LLM step shows only "the provider never became ready", without its cause or the screen that ended it.
Every shuttle start sleeps ten seconds between startup probes although the provider is ready within about a second.
A seats chair held behind a busy advisor waits up to 20 minutes before its stall is noticed.
The cards fix these in order:
- the Claude box reading;
- the start error on `Result`;
- the never-ready pane tail;
- the startup probe cadence with its dismissal throttle;
- the hold-deferral cap.

## Card Index

1 — claude-lone-eol-mark-box — Claude's input-box reading treats a box holding only `%` as empty, and the pane-text-injection background names it
2 — start-error-on-result — `Result.StartError` carries a not-started run's start error, and the single-LLM producer's failure appends it
3 — never-ready-pane-tail — a never-ready, no-cause startup teardown's error ends with the last lines of its last startup capture
4 — startup-probe-backoff-and-dismissal-gap — `awaitStartup` probes every poll interval with backoff to the dismissal gap, and gate dismissals are spaced one gap apart
5 — hold-deferral-ten-minutes — `holdNoticeMaxDeferral` drops from 20 to 10 minutes

## Shared Decisions

- The record's "Added after Discussion" decision supersedes D5's exemption.
  `awaitStartup` follows `PATTERN-daemon-wakeups`' backoff rule, and that background is not amended, although the record's Constraints line still describes the superseded amendment.
- Every timing test injects the run clock and never waits out an interval (`PATTERN-test-speed`).

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
