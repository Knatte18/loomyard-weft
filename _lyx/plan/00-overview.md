---
format: 5
approved: false
language: go
---

# Plan: shuttle-startup-ready

Shuttle starts stop failing on a ready Claude pane whose input line shows a stray `%`, a not-started producer failure names its cause and the pane's last lines, every shuttle start passes its startup probe within about a second of the provider becoming ready, and a seats chair's held-turn notice is deferred no longer than 10 minutes.
Each card compiles and passes its own tests; card 5 lands the dismissal gap before card 6 raises the probe cadence, so no commit probes faster than it spaces key presses.

## Card Index

1 — claude-lone-percent — Claude's input-box reading treats a lone `%` as empty, and the pane-text-injection background says so
2 — hold-deferral-ten-minutes — `holdNoticeMaxDeferral` drops from 20 to 10 minutes
3 — never-ready-pane-tail — a never-ready start's error ends with the last lines of its last startup capture
4 — start-error-on-result — `Result.StartError` carries a not-started run's cause into the single-LLM producer's message
5 — gate-dismissal-gap — a startup-gate dismissal replays at most once per `poll_interval_ms × liveness_every_n_polls`
6 — startup-probe-backoff — `awaitStartup` probes from the poll interval and backs off up to the dismissal gap while the pane is unchanged

## Shared Decisions

- The parent's addendum to the decision record supersedes its D5 and its `PATTERN-daemon-wakeups` constraint line: `awaitStartup` follows the backoff rule and `pattern/PATTERN-daemon-wakeups.md` is not amended.
- `PATTERN-refusal-way-forward` does not apply: no new refusal is added.
- `PATTERN-test-speed`: the cadence and throttle rows run on the fake clock and never wait out an interval.

## verify:

```
go build ./...
go vet -tags integration ./internal/shuttleengine/... ./internal/shedadapters/... ./cmd/lyx/...
go vet -tags tmux ./internal/shuttleengine/... ./internal/shedadapters/... ./cmd/lyx/...
go vet -tags llm ./internal/shuttleengine/... ./internal/shedadapters/... ./cmd/lyx/...
go test ./internal/shuttleengine/... ./internal/shedadapters/... ./cmd/lyx/...
go test -tags integration ./internal/shuttleengine/... ./internal/shedadapters/... ./cmd/lyx/...
```
