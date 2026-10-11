---
format: 5
approved: false
language: go
---

# Plan: shuttle-startup-ready

Shuttle starts stop failing on a ready Claude pane whose input line shows a stray `%`, a never-started producer failure names its cause and the pane's last lines, every shuttle start passes its startup probe within about a second of the provider becoming ready, and a seats chair's held-turn notice is deferred no longer than 10 minutes.
Each card is one decision group of the decision record and builds and tests on its own.

## Card Index

1 — claude-lone-percent — Claude's input-box reading treats a lone `%` as an empty box, the invariant's background says so, and a start test pins the color send passing the guard
2 — start-error-on-result — `Result.StartError` carries a never-started run's start error into the single-LLM producer's failure message
3 — startup-teardown-pane-tail — a never-ready start's error ends with the last lines of its last startup capture
4 — startup-probe-cadence — `awaitStartup` probes at the poll interval with a backoff, and a startup-gate dismissal is replayed at most once per dismissal gap
5 — hold-deferral-ten-minutes — `holdNoticeMaxDeferral` drops to 10 minutes

## Shared Decisions

- The parent's amendment supersedes D5's exemption reading: `awaitStartup` follows the `PATTERN-daemon-wakeups` backoff rule and `pattern/PATTERN-daemon-wakeups.md` is not amended.
- D4's tail covers `awaitStartup`'s never-ready exits that carry no cause; a teardown with a cause adds nothing, and the skill-load death is a nil cause that is not never-ready.
- The decision record's `PATTERN-daemon-wakeups` constraint line, which says `awaitStartup` gets no backoff, is superseded by the parent amendment the record also carries; the plan follows the amendment.
- Any play that sent an input starts D6's dismissal gap; only an empty sequence or a play whose first input failed leaves the next probe free to dismiss.
- The `%` knowledge lives only in `claudeengine` (`PATTERN-shuttle-provider-seam`); the only `PATTERN` background touched is `pattern/PATTERN-pane-text-injection.md`.
- Card 1 raises the `internal/shuttleengine/claudeengine` row of `cmd/lyx/testdata/test-budget.yaml` for its one new test; the new tests of cards 2 to 5 sit within their packages' rows.

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
