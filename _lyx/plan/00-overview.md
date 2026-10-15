---
format: 5
approved: false
language: go
---

# Plan: lyx report timing

`lyx report timing` prints, mechanically from the hub's run records, where run time went: per run, per phase, per day or week, with machine sleep and waits apart, and who settled each stop.
It is a second compute entry in `internal/reportengine` over `convergence`'s snapshot and run selection.
To tell the orch, the LLM driver, the operator and the Go driver apart, the goto and step verbs record the caller's label on the shed history.
Two small moves come first: the duration formatters into `internal/durfmt` and the journal sleep parser into `internal/sleepjournal`, and the settle-record format into `internal/shedrun`.

## Card Index

1 — durfmt — move the pane duration formatters into a stdlib-only leaf
2 — sleepjournal — move the systemd-journal sleep parser into a stdlib-only leaf
3 — settle-records-shedrun — move the settle-record format and its tolerant decoder into shedrun
4 — caller-label — classify a caller's strand into a history label
5 — history-caller-field — record the caller label on goto entries and resumed entries
6 — caller-label-wiring — pass the label from the goto and step verbs, the Go driver and batten
7 — pair-branch-snapshot — snapshot the pairs' records branches and the snapshot time
8 — timing-spans-phases — compute spans, per-row phases, sleep, buckets and medians
9 — timing-waits-attempts — add waits and arbiter and alignment attempt time
10 — timing-stops-settlers — add stops and who settled each
11 — timing-markdown — render the timing report as markdown
12 — report-timing-verb — add the lyx report timing verb

## Rename mechanic

1. Run `git mv <old> <new>` FIRST, before any other change to the moved file.
2. Then make ONLY surgical edits (package declaration, imports, identifier
   retargeting) — no unrelated rewrites.
3. A genuinely new file with no predecessor belongs in a separate `Create` card, never folded
   into the `Rename` pair.
4. Never write the relocated file from scratch and delete the original — that loses
   git history exactly as an unstructured create+delete pair would.

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
