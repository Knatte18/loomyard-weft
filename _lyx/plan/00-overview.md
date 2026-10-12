---
format: 5
approved: false
language: go
---

# Plan: shuttle-startup-ready

Shuttle starts stop failing on a ready Claude pane whose input line shows a stray `%`, a not-started producer failure names its cause and the pane's last lines, every shuttle start passes its startup probe within about a second of the provider becoming ready, and a seats chair's held-turn notice is deferred no longer than 10 minutes.
The cards follow the decision record's decisions D1 to D7 and its two post-Discussion amendments (the `awaitStartup` backoff under `PATTERN-daemon-wakeups`, and D4's never-ready-with-no-cause tail scope).

## Card Index

1 — claude-lone-percent-box — Claude's input-box reading treats a lone `%` as empty, with the `PATTERN-pane-text-injection` background line
2 — hold-deferral-ten-minutes — `holdNoticeMaxDeferral` drops from 20 to 10 minutes
3 — result-start-error — `Result.StartError`, set only on `RunGated`'s not-started branch
4 — singlellm-start-error — the single-LLM producer's not-started failure appends the start error
5 — never-ready-pane-tail — a never-ready, no-cause teardown's error ends with the pane's last lines
6 — startup-probe-cadence — `awaitStartup` probes every poll interval with a backoff, and a gate dismissal is replayed once per gap

## Shared Decisions

- `PATTERN-shuttle-provider-seam`: the `%` knowledge lives only in `claudeengine`.
- `PATTERN-test-speed`: the cadence and throttle tests inject the run clock and never wait out an interval.
- `PATTERN-test-economy`: each scenario extends an existing table or test; the only new top-level tests are one in `claudeengine` (budget row raised in card 1) and one in `shuttleengine` (within its budget row).
- `PATTERN-comment-line-breaks`: every rewritten or new Go comment keeps one sentence per line.
- `PATTERN-refusal-way-forward` does not apply: no new refusal is added.

## verify:

```
go build ./...
go vet -tags integration ./internal/shuttleengine/... ./internal/shedadapters/... ./internal/loomcli/... ./cmd/lyx/...
go vet -tags tmux ./internal/shuttleengine/... ./internal/shedadapters/... ./internal/loomcli/... ./cmd/lyx/...
go vet -tags llm ./internal/shuttleengine/... ./internal/shedadapters/... ./internal/loomcli/... ./cmd/lyx/...
go test ./internal/shuttleengine/... ./internal/shedadapters/... ./internal/loomcli/... ./cmd/lyx/...
go test -tags integration ./internal/shuttleengine/... ./internal/shedadapters/... ./internal/loomcli/... ./cmd/lyx/...
```
