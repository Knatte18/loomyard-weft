---
format: 5
approved: false
language: go
---

# Plan: shuttle-startup-ready

Shuttle starts stop failing on a ready Claude pane whose input line shows a stray `%`, a not-started single-LLM producer failure names its cause and the pane's last lines, every shuttle start passes its startup probe about a second after the provider redraws ready, and a seats chair's held-turn notice is deferred no longer than 10 minutes.
The cards follow the decision record's D1–D7 and its three after-Discussion additions.

## Card Index

1 — claude-lone-percent-box — Claude's box reading counts a lone `%` as an empty box, and the pane-text-injection background says so (D1, D2).
2 — hold-deferral-ten-minutes — `holdNoticeMaxDeferral` drops to 10 minutes, pinned by the existing deferral row (D7).
3 — result-start-error — `Result.StartError` carries the start error on `RunGated`'s not-ready branch only (D3).
4 — single-llm-start-error — The single-LLM producer appends `StartError` to its not-started failure, still wrapping `ErrNotStarted` (D3).
5 — never-ready-pane-tail — A never-ready teardown's error ends with the last startup capture's last lines (D4).
6 — startup-probe-cadence — `awaitStartup` probes from one poll interval with the daemon-wakeups backoff, and gate dismissals are spaced one dismissal gap apart (D5, D6).

## Shared Decisions

- The after-Discussion addition "awaitStartup backs off under PATTERN-daemon-wakeups" supersedes D5's exemption and the Constraints line that says D5 amends `pattern/PATTERN-daemon-wakeups.md`: card 6 applies the backoff, and no card amends that background.
- The acceptance criterion "a start whose pane shows a ready box with a lone `%` passes the color send and the skill-load send" is carried by card 1's classifier rows, not a start-level test: shuttle reads Claude's box only through `IdleSession` and `InputBoxText`, and no shuttle test runs the real Claude engine, so a start-level test would need a new real-engine harness for no behavior those rows leave uncovered.
- D5 and D6 share card 6: the one-second probe is safe only with the dismissal spacing in the same commit.
- The `lyx:carry-over` entries in the record's Open risks are not planned as scope; each names a point the after-Discussion additions already restate, and the cards follow those restatements.

## verify:

go build ./... && go vet -tags integration ./... && go vet -tags tmux ./... && go vet -tags llm ./... && go test ./... && go test -tags integration ./...
