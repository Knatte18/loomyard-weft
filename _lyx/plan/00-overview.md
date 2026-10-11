---
format: 5
approved: true
language: go
---

# Plan: small-fixes-oct11

Five small, independent fixes from the board entry `small-fixes-oct11`, one decision each in the decision record.
Tokencount stops counting a crash or stall gap as working time (D1) and counts only a reused slug's own run's transcripts (D2).
`lyx loom step` and `lyx loom run` help and refusals name the driverless seed beside `lyx loom start`, and the bootstrap's disagreeing-seed refusal names a way forward that does not loop (D3, D4).
The plan review rubric points at the target-redundancy checks (D5, issue #542), and three batten notices lead with the child's state again (D6).
No card changes what `lyx loom step` or `lyx loom run` do, and every package's new coverage extends existing tests except card 3's one help test.

## Card Index

1 — tokencount-turn-start-and-stall-cap — an interval into a turn-start line, or over the 30-minute stall cap, is idle
2 — tokencount-run-lower-bound — a run counts only transcripts that start after the slug's previous run, with a report line for those left out
3 — loom-help-seeding-routes — step and run help name `lyx loom start` and the driverless `lyx shed seed`, then `lyx loom step`
4 — loom-missing-seed-remedy — the missing-seed refusal for run and step, and run's status-file refusal, name the driverless route
5 — loom-disagreeing-seed-way-forward — the bootstrap's disagreeing-seed refusal names `rm` of the seed and a re-seed with the recorded parent
6 — plan-rubric-redundancy-pointer — the plan review rubric's Proposed fixes section names the target-redundancy checks
7 — batten-notice-state-lead — quiet, api-error and driver-dead notices start with the child's state and row

## Shared Decisions

- Every package this plan touches sits at its row in `cmd/lyx/testdata/test-budget.yaml`, so coverage extends existing tests and tables; card 3's help test is the one new top-level test, and it raises `internal/loomcli`'s row.
- Tokencount scenarios stay in the untagged tier on fixed timestamps, except card 2's end-to-end case in the existing integration test over a real archive-tag repository (`PATTERN-test-speed`).
- Every refusal whose way forward changes lands with its `contracts/specs/refusal-spec.md` row and a test that reaches it (`PATTERN-refusal-way-forward`), and names only `lyx loom start`, `lyx loom step`, `lyx shed seed` and a plain `rm <path>` (`PATTERN-no-denied-recovery`).
- Docs change with their code: tokencount's package doc in `main.go` (cards 1 and 2), battenshed's `doc.go` (card 7) and the `docs/overview.md` batten `step` sentence (card 3).

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
