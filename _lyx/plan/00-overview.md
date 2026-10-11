---
format: 5
approved: false
language: go
---

# Plan: shuttle-startup-ready

Shuttle starts stop failing on a ready Claude pane whose input line shows a stray `%`, a not-started single-LLM producer failure names its cause and the pane's last lines, every shuttle start passes its startup probe within about a second of the provider becoming ready, and a seats chair's held-turn notice is deferred no longer than 10 minutes.
The decision record's later additions supersede D5's exemption with a backoff under `PATTERN-daemon-wakeups`, which stays unamended, scope D4's tail to never-ready teardowns with no cause, and start the dismissal gap on any play that sent a key.

## Card Index

1 — claude-lone-eol-mark — Claude's box reading treats a lone `%` as empty; the pane-text-injection background records it
2 — run-gated-start-error — `Result.StartError` carries a not-started run's cause into the single-LLM producer's message
3 — never-ready-pane-tail — a never-ready, cause-free startup teardown's error ends with the pane's last lines
4 — startup-probe-cadence — `awaitStartup` backs off from one second to the dismissal gap, and gate dismissals keep that gap
5 — hold-deferral-cap — `holdNoticeMaxDeferral` drops to 10 minutes

## Shared Decisions

- The `%` knowledge lives only in `internal/shuttleengine/claudeengine`, per `PATTERN-shuttle-provider-seam`.
- Every negative startup answer still goes through `checkLivenessTick`, `classifyStartupWindow`, `classifyDeadlineExpiry` or a direct `allOutputFilesExist`, per `PATTERN-completion-signal`; card 4 changes cadence and key replay only.
- Cadence and throttle tests inject the run clock and never wait out an interval, per `PATTERN-test-speed`.

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
