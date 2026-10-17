---
format: 5
approved: false
language: go
---

# Plan: pr-gate-run-identity

A loom or darn run's landing rows act only on what belongs to the run itself (issue #692).
Every pull-request lookup of Publish, PR-Gate, PR-Align, Finalize and the `lyx loom approve` and `lyx loom reject` verbs is bounded by the run's start, the `started_at` stamp of the pair's seed, so a reused slug never adopts an earlier run's pull request, and Publish stops with a way forward on an older open one it cannot replace.
The alignment agent's decision snapshot counts a board note only when it is titled `Decision for <slug>:`, a form `internal/alignment` declares once and the orch role stencil renders, and the snapshot's escalation reason is declared once.

## Card Index

1 — decision-reason-once — declare the decision-recorded reason once in internal/alignment and retarget both consumers
2 — snapshot-title-form — declare the decision-note title form and count only notes in it toward the snapshot
3 — orch-role-title-form — render the title form into the orch role stencil as a required marker
4 — seed-run-start — read the run's start bound from the pair's seed in loomcli
5 — run-bounded-lookup — bound FindPullRequest and all its callers by the run's start, wired from the seed
6 — publish-older-open-stop — stop Publish on an older open pull request with a way forward

## verify:

```bash
go build ./...
go test ./internal/alignment ./internal/landingshed ./internal/loomcli ./internal/orchengine ./contracts/stencils
go test -tags integration ./internal/alignment ./internal/landingshed ./internal/loomcli ./internal/orchengine ./contracts/stencils
```
