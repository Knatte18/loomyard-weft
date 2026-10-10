# Gate test scope: which gate runs which tier, bounded hub-wide

`gate-test-scope` · Task · bug, enhancement, webster, loom, stencil, config, burler · paused · Plan-Bouncer

Depends on: [`plan-gate-checks`](design-plan-gate-checks.md)

## Seen

Before the 2026-10-08 crash, Webster implementer forks ran `go build ./... && go test ./...` plus `go test -tags integration` on their packages every 20 to 60 seconds.
That is the card gate command `cardGateCommand` derives (`internal/websterengine/cardgate.go`), run once per card.
Gates, forks, the orch and hub fix agents built and tested with no hub-wide bound, and with four runs in flight the user slice reached its 16 GiB `MemoryHigh` and the machine froze.
From `test-tiers-hardening`: the Webster-Burler round gate compiles the `tmux` tier and never runs it, so a tier 3 defect surfaces only at Publish; a task can empty its own `publish_verify` in the pair's `landing.yaml`; no gate runs the `llm` tier.

## Goal

- The card gate runs tier 1 for the card's own packages only: no `go build ./... && go test ./...` over the module and no tier 2.
- Tier 2 runs once per batch for the batch's packages, and in full at the plan verify, Publish and Finalize.
- Tier 3 stays at Publish; Discussion decides, against its cost, whether the round gate runs an impacted-set tier 3 pass.
- The `llm` tier stays by hand, and `pattern/PATTERN-test-speed.md` says so as a decision.
- A hub-wide slot lock (an OS file lock per slot, slot count in config) bounds concurrent gate test runs, each run caps `-p`, and the wait is logged and shown in status.
- The webster stencils tell each fork which packages it may test, and spawned agents are refused a raw full-module `go test` or `go build ./...`.
- A pair's own config cannot empty `publish_verify`.
- `pattern/PATTERN-test-speed.md`'s gate table and CLAUDE.md's push rule match the new gates in the same commit.
- The comment lint (`lyx loom lint-comments`, `internal/loomcli/lintcomments.go`) breaks only at a comma before a coordinating conjunction whose clause has its own subject and verb, not at every comma-plus-conjunction, so reviewers stop judging the clause-boundary rule by hand.
- A Burler reviewer's range covers what the gate failed on: when Publish failed on a defect that arrived through the merge of main, the round after a goto reads that merge and the failure, not only `git diff $(git merge-base main HEAD)..HEAD`.
- Build on the card gate as `plan-gate-checks` left it: keep its `go -C <module>` split for nested modules when narrowing to the card's own packages, and update `cardgate_test.go`.
- The plan gate's `caller-uncovered` check now runs `go list` with export data (up to 3 minutes, cgo grammars on a cold cache) on every plan-gate pass; it takes a slot of the hub-wide lock like any other gate build.
