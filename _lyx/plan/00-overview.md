---
format: 5
approved: false
language: go
---

# Plan: shuttle-startup-ready

Shuttle starts stop failing on a ready Claude pane whose input box shows a stray `%`, a not-started single-LLM producer failure names its cause and the pane's last lines, every shuttle start sees a ready provider within about a second, and a seats chair's held-turn notice is deferred no longer than 10 minutes.
The work sits in `internal/shuttleengine`, its Claude provider and the single-LLM producer in `internal/shedadapters`.

## Card Index

1 — claude-lone-eol-mark — Claude's box reading treats a lone `%` as an empty box, recorded in the pane-text-injection background
2 — not-started-cause — `Result.StartError` carries a not-started run's start error into the single-LLM producer's failure message
3 — never-ready-pane-tail — a never-ready start's error ends with its last startup capture's last lines
4 — startup-probe-cadence — `awaitStartup` backs off from one poll interval to the dismissal gap, and gate dismissals are spaced one gap apart
5 — hold-deferral-cap — `holdNoticeMaxDeferral` drops from 20 to 10 minutes

## Shared Decisions

- The `%` knowledge lives only in `claudeengine` (`PATTERN-shuttle-provider-seam`).
- The dismissal gap is `pollInterval(cfg)` times `LivenessEveryNPolls`, floored to 1 as `Wait` floors it; card 4 both caps the startup backoff and spaces dismissals with it.
- No return statement is added to or removed from `awaitStartup` or `abandonStartup`, so the completion-signal enforcement scan's counts stay as they are.

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
