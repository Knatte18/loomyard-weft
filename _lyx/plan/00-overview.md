---
format: 5
approved: false
language: go
---

# Plan: shuttle-startup-ready

Shuttle starts stop failing on a ready Claude pane whose input box shows a stray `%`, a not-started single-LLM failure names its cause and the pane's last lines, every shuttle start sees a ready provider within about a second, and a seats chair's held-turn notice is deferred no longer than 10 minutes.
The decision record's later parent additions govern: `awaitStartup` backs off under `PATTERN-daemon-wakeups` as written instead of D5's exemption, so the background file is not amended; D4's tail goes on never-ready exits with no cause; and any dismissal play that sent a key starts the dismissal gap.
The carried-over Discussion-Review findings are each answered by those additions and are not planned as scope.

## Card Index

1 — claude-lone-percent-box — Claude's box reading treats a box holding only `%` as empty, and the pane-text-injection background says so
2 — lone-percent-start — a start on a ready Claude box holding `%` passes its color and skill-load sends
3 — result-start-error — `Result.StartError` carries a not-started run's error into the single-LLM producer's failure
4 — never-ready-pane-tail — a never-ready start's error ends with its last startup capture's last lines
5 — startup-probe-cadence — the startup probe backs off from one poll interval to the dismissal gap, and dismissals are spaced by that gap
6 — hold-deferral-cap — `holdNoticeMaxDeferral` drops to 10 minutes

## Shared Decisions

- The `%` knowledge lives in `claudeengine` alone (`PATTERN-shuttle-provider-seam`); `shuttleengine` stays provider-neutral.
- Timing tests inject the run clock and never wait out an interval (`PATTERN-test-speed`); new scenarios are rows in existing tables where one covers the surface (`PATTERN-test-economy`).
- No refusal is added, so `PATTERN-refusal-way-forward` does not apply.

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
lyx gate test ./...
lyx gate test --tags integration ./...
```
