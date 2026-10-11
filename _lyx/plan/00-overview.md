---
format: 5
approved: false
language: go
---

# Plan: shuttle-startup-ready

Shuttle starts stop failing on a ready Claude pane whose input box shows a stray `%` (issue #549).
A not-started single-LLM producer failure names its cause and, for a never-ready start, the pane's last lines.
The startup probe backs off from `poll_interval_ms` instead of sleeping the whole liveness interval, with startup-gate dismissals spaced by that interval.
A seats chair's held-turn notice is deferred at most 10 minutes.

## Card Index

1 — claude-lone-eol-mark — Claude's input-box reading treats a box holding only `%` as empty, and the pane-text-injection background records it
2 — never-ready-error-tail — a never-ready start's error with no cause ends with the last lines of its last startup capture
3 — not-started-start-error — `Result.StartError` carries the start error to the single-LLM producer's not-started failure
4 — startup-probe-cadence — `awaitStartup` backs off from the poll interval up to the dismissal gap, and a gate dismissal is replayed at most once per gap
5 — hold-max-deferral — `holdNoticeMaxDeferral` drops from 20 to 10 minutes

## Shared Decisions

- Provider knowledge of the `%` mark stays in `claudeengine` (`PATTERN-shuttle-provider-seam`).
- Every negative startup answer keeps going through the existing completion-signal primitives (`PATTERN-completion-signal`); no card adds a startup exit.
- Timing tests inject the run clock and never wait out an interval (`PATTERN-test-speed`).

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
