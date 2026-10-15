---
format: 5
approved: true
language: go
---

# Plan: refusal-route-config-empty

Two bundled fixes (issues #625 and #632).
Begin-batch's dispatch refusal stops calling itself `plan_drifted`: it carries `plan_conflict` with a cause, leaves a Go-written conflict record, and under loom a Webster stuck outcome with a new conflict reroutes to Plan-Burler-Review instead of halting, while a repeated one halts with a hand-edit way forward.
`lyx config`'s editor paths refuse without a terminal, and an empty editor result is never saved over a layer.

## Card Index

1 — plan-conflict-class — begin-batch's blocking dispatch refusal becomes PlanConflictError with a plan_defect or tree_moved cause and a plan_conflict envelope
2 — conflict-record — begin-batch writes and removes the never-tracked conflict record, and Run removes it on entry
3 — webster-reroute-edge — the Webster row gains on_reroute to Plan-Burler-Review, whose instructions name the record, and the cap clause becomes row-neutral
4 — adapter-routes-conflict — the Webster adapter reroutes a stuck outcome with a new conflict and halts on a repeated one
5 — config-empty-edit-abort — EditPath aborts on an empty or whitespace-only editor result and restores the previous bytes
6 — config-terminal-guard — bare lyx config and lyx config menu refuse when stdin is not a terminal

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
