---
format: 5
approved: false
language: go
---

# Plan: shuttle-startup-ready

Shuttle starts stop failing on a ready Claude pane whose input box shows a stray `%`.
A not-started single-LLM failure names its cause and the pane's last lines.
The startup probe sees a ready provider within about a second, backing off under `PATTERN-daemon-wakeups` while spacing gate dismissals by the old cadence.
A seats chair's held-turn notice is deferred no longer than 10 minutes.

## Card Index

1 — claude-lone-eol-mark — Claude's box reading treats a lone `%` as empty, and the pane-text-injection background says so
2 — start-error-on-result — `Result.StartError` carries a not-started start's error to the single-LLM producer's failure message
3 — never-ready-pane-tail — a never-ready, cause-free startup teardown's error ends with the last capture's lines
4 — startup-probe-cadence — `awaitStartup` backs off from the poll interval to the dismissal gap, and a gate dismissal is played at most once per gap
5 — hold-deferral-10m — `holdNoticeMaxDeferral` drops to 10 minutes

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
