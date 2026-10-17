# pr-gate-run-identity: run-bounded pull-request lookups and a single decision-note form

Both batches recorded `status: done`, with no departures.

## Batch 01-decision-reason-once (head 70b02b6fc41ace882ef4e6e3b03263913e06c3a1)

Cards 1 to 3.
`internal/alignment` now declares the decision-recorded reason once, and PR-Gate and parent review use it.
It also declares the decision-note title form `Decision for <slug>:`, and the decision snapshot counts only board notes in that form.
The orch role stencil renders the title form as a required marker.
Scope guard deviations: `internal/landingshed/pralign.go`, `internal/loomcli/decision.go`.

## Batch 04-seed-run-start (head a5a44959939aaf192ebd0cadff0f3d6318f129ec)

Cards 4 to 6.
`loomcli` reads the run's start from the pair's seed.
`FindPullRequest` is bounded by that start and its callers are wired from the seed, including the approve and reject verbs.
Publish stops with a way forward on an older open pull request it cannot replace.
Scope guard deviations: `internal/loomcli/approve_test.go`, `internal/loomcli/runstart.go`.
