---
format: 5
approved: false
language: go
---

# Plan: shuttle-startup-ready

Shuttle starts stop failing on a ready Claude pane whose input line shows a stray `%`, a not-started producer failure names its cause and the pane's last lines, every shuttle start passes its startup probe within about a second of the provider becoming ready, and a seats chair's held-turn notice is deferred no longer than 10 minutes.
The five cards follow the decision record's order of dependence: the provider's box reading, the start error's tail, the start error on `Result`, the startup cadence with its dismissal spacing, and the deferral cap.

## Card Index

1 — claude-lone-percent — claudeengine reads a box holding only `%` as empty; the injection background names it
2 — startup-error-tail — a never-ready teardown's error ends with the last lines of its last startup capture
3 — result-start-error — `Result.StartError` carries the start error into the single-LLM failure message
4 — startup-probe-cadence — `awaitStartup` probes at the poll interval with backoff, and a gate dismissal is replayed at most once per gap
5 — hold-deferral-ten-minutes — `holdNoticeMaxDeferral` drops from 20 to 10 minutes

## Shared Decisions

- The decision record's added decision supersedes D5's exemption reading: `awaitStartup` follows the `PATTERN-daemon-wakeups` backoff rule, so `PATTERN-daemon-wakeups` is not amended, and the Constraints line that says it is amended and that `awaitStartup` gets no backoff is not planned.
- `completionsignal_enforcement_test.go` audits the negative-verdict return sites and `allOutputFilesExist` call sites in `wait.go`; cards 2 and 4 add none, so its audited sets stay as they are.
- The review carry-over entry in the decision record's open risks is not scope.

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./internal/shuttleengine/... ./internal/shedadapters/... ./internal/loomcli/... ./internal/websterengine/... ./cmd/lyx/...
go test -tags integration ./internal/shuttleengine/... ./internal/shedadapters/... ./internal/loomcli/... ./internal/websterengine/... ./cmd/lyx/...
```
