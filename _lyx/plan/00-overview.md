---
format: 5
approved: false
language: go
---

# Plan: shuttle startup ready

Shuttle starts stop failing on a ready Claude pane whose input line shows a stray `%`.
A not-started single-LLM failure names its cause and the pane's last lines.
Every start passes its startup probe within about a second of the provider becoming ready, with gate dismissals still spaced by the dismissal gap.
A seats chair's held-turn notice is deferred no longer than 10 minutes.

## Card Index

1 — claude-lone-eol-mark — Claude's box reading treats a lone `%` as empty, and the pane-injection background records it
2 — start-error-on-result — `Result.StartError` carries a not-started run's start error into the single-LLM failure
3 — never-ready-pane-tail — a never-ready, cause-free startup teardown's error ends with the last startup capture's lines
4 — startup-probe-backoff — `awaitStartup` probes from the poll interval with capture-reset backoff, and dismissals are spaced by the dismissal gap
5 — hold-deferral-ten-minutes — `holdNoticeMaxDeferral` drops to 10 minutes

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
