---
format: 5
approved: false
language: go
---

# Plan: shuttle-startup-ready

Shuttle starts stop failing on a ready Claude pane whose input box shows a stray `%` (issue #549).
A not-started single-LLM producer failure names its cause and the pane's last lines.
Every shuttle start passes its startup probe within about a second of the provider becoming ready, with startup-gate keys still spaced by the dismissal gap.
A seats chair's held-turn notice is deferred no longer than 10 minutes.

## Card Index

1 — claude-lone-eol-mark — Claude's box readings treat a box holding only `%` as empty, and the pane-text-injection background states it
2 — start-error-on-result — `Result.StartError` carries a not-started run's start error into the single-LLM producer's failure
3 — never-ready-pane-tail — a never-ready, no-cause startup teardown's error ends with the last capture's last lines
4 — startup-probe-cadence — `awaitStartup` backs off from the poll interval to the dismissal gap, and gate dismissals are at least one gap apart
5 — hold-deferral-ten-minutes — `holdNoticeMaxDeferral` drops from 20 to 10 minutes

## Shared Decisions

- `PATTERN-shuttle-provider-seam`: the `%` knowledge lives only in `claudeengine`; `shuttleengine` gains no Claude specifics.
- `PATTERN-completion-signal`: every negative startup answer still goes through `checkLivenessTick`, `classifyStartupWindow`, `classifyDeadlineExpiry` or a direct `allOutputFilesExist`; cards 3 and 4 add no exit and keep `completionsignal_enforcement_test.go`'s counts.
- `PATTERN-daemon-wakeups`: the startup probe follows the backoff rule with a named ceiling, `dismissalGap`; the background is not amended and no sub-second literal is added.
- `PATTERN-test-speed`: cadence and throttle tests inject the run clock and never wait out an interval.

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
