---
format: 5
approved: false
language: go
---

# Plan: Shuttle startup ready

Shuttle starts stop failing on a ready Claude pane whose input box shows a stray `%`, a not-started single-LLM producer failure names its cause and the pane's last lines, every start passes its startup probe within about a poll interval of the provider becoming ready, and a seats chair's held-turn notice is deferred at most 10 minutes.
The `%` reading lives in `claudeengine` alone; the startup cadence, the dismissal gap, the never-ready tail and `Result.StartError` live in `shuttleengine`; the producer message in `shedadapters`.

## Card Index

1 — claude-lone-percent-box — Claude's box reading treats a lone `%` as an empty box, and the pane-text-injection background states it
2 — never-ready-pane-tail — awaitStartup's cause-less not-ready teardowns end their error with the last startup capture's last lines
3 — start-error-on-result — RunGated's not-started Result carries the start error, and the single-LLM producer's failure shows it
4 — startup-probe-every-poll — awaitStartup probes every poll interval, and a gate dismissal replays at most once per dismissal gap
5 — hold-deferral-ten-minutes — holdNoticeMaxDeferral drops from 20 to 10 minutes

## Shared Decisions

- The tail wording is the send failure's: `. The pane's last lines:\n` followed by `paneTailLines` non-blank lines, cut by the one helper `captureTail`.
- D5 and D6 share card 4: neither lands without the other, since probing every second without the throttle replays a dismissal on every probe.
- No config key is added; the dismissal gap is `poll_interval_ms × liveness_every_n_polls`, floored as `Wait` floors it.

## verify:

```bash
go build ./...
go vet -tags integration ./internal/shuttleengine/... ./internal/shedadapters/... ./cmd/lyx/...
go vet -tags tmux ./internal/shuttleengine/... ./internal/shedadapters/... ./cmd/lyx/...
go vet -tags llm ./internal/shuttleengine/... ./internal/shedadapters/... ./cmd/lyx/...
go test ./internal/shuttleengine/... ./internal/shedadapters/... ./cmd/lyx/...
go test -tags integration ./internal/shuttleengine/... ./internal/shedadapters/... ./cmd/lyx/...
```
