---
format: 5
approved: false
language: go
---

# Plan: shuttle startup ready

Shuttle starts stop failing on a ready Claude pane whose input box shows a stray `%`, a not-started single-LLM failure names its cause and the pane's last lines, every shuttle start passes its startup probe within about a second of the provider becoming ready, and a seats chair's held-turn notice is deferred no longer than 10 minutes.
The cards follow the decision record's D1 to D7 and the parent's three post-Discussion additions.

## Card Index

1 — claude-lone-eol-mark — Claude's box reading treats a lone `%` as an empty box, and the pane-text-injection background names it (D1, D2).
2 — hold-deferral-ten-minutes — `holdNoticeMaxDeferral` drops to 10 minutes (D7).
3 — not-started-start-error — `Result.StartError` carries the error of a start `RunGated` folds into a not-started result (D3).
4 — single-llm-start-error — the single-LLM producer's not-started failure appends `Result.StartError` (D3).
5 — never-ready-pane-tail — a never-ready start with no cause ends its error with the last startup capture's last lines (D4).
6 — startup-probe-cadence — `awaitStartup` probes at the poll interval with backoff, and gate dismissals are spaced by the dismissal gap (D5, D6).

## Shared Decisions

- The parent's addition "awaitStartup backs off under PATTERN-daemon-wakeups" supersedes D5's exemption and the Constraints line saying D5 amends `pattern/PATTERN-daemon-wakeups.md`: no card amends that background.
- The parent's addition restating D4's scope governs Card 5: the tail goes only on never-ready exits with no cause.
- The parent's addition "any play that sent a key starts the dismissal gap" governs Card 6's throttle.

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
