---
format: 5
approved: false
language: go
---

# Plan: alignment agent at parent review and PR-Gate

A cold-start alignment agent, spawned by Go, replaces the orch wake at loom's parent review and at PR-Gate.
It reads the board, the other in-flight runs' records through a new `lyx loom peers` verb and the parent's landed commits.
It settles the review itself (approve, or reject on a scope deviation) unless it finds a substantive overlap or a question it cannot settle, which it escalates to the run's parent.
Go applies its verdict through the stores the existing verbs write, and every failure falls back to today's path.
The cards add the config keys, the peers verb, the `internal/alignment` package and its session, the parent-review gate and its wiring, the shared approval write, the `PR-Align` producer and row, and the orch stencil.

## Card Index

1 — loom-alignment-config — loom.yaml's four alignment keys, their layer classes, the timeout-below-wait check and ResolveAlignment
2 — fabric-commit-log — Fabric.CommitLog lists a range's commits as short SHA and subject
3 — loom-peers — `lyx loom peers` and `peers show` read the in-flight runs' records and the parent's commits
4 — alignment-records — internal/alignment: role, points, verdict schema, attempts, prompt, spec, snapshot, and its stencil
5 — alignment-session — alignment.Session spawns once per key, polls without blocking, closes and marks applied
6 — parent-review-gate-alignment — the parent-review gate spawns the agent first, applies its verdict and falls back to delivery
7 — parent-review-alignment-wiring — loom wires the gate's Aligner, the decision snapshot and the brief's alignment section
8 — shared-approval-write — RecordApproval and RecordRejection become the one write path the verbs use
9 — pr-align-producer — the PR-Align producer applies the agent's verdict, and PR-Gate names an escalation in its notice
10 — pr-align-row — loom's recipe runs PR-Align between Publish and PR-Gate
11 — orch-stencil-alignment — the orch role and CLAUDE.md for escalation-only reviews and recorded decisions

## Shared Decisions

### Decision: peers-geometry

`lyx loom peers` resolves the caller's location through `lyxcwd`, in its own pre-run as the `review` and `decision` groups do, not through `internal/cliwire`.
D6 names `internal/cliwire`, but that package serves only the standalone-capable CLIs and may not import `lyxcwd` (`internal/cliwire/doc.go`).
Every `lyx loom` verb is hub-only and resolves through `lyxcwd.Resolve` (`internal/loomcli/cli.go`'s header), so the verb keeps loom's resolution, and `PATTERN-cliwire-sole-wiring` does not apply to a hub-only CLI.
The rest of D6 holds as written: a peer's pair path comes from `fabricengine.WorktreePath`, and every git read goes through `internal/fabricengine`.

### Decision: pr-align-in-landingshed

`PR-Align` is a `landingshed` producer beside `PR-Gate`, since it reads the pull request through the same `FindPullRequest`, writes the records `PR-Gate` reads, and shares `Deps`.
`internal/alignment` owns the agent; `landingshed` reaches it through the `AlignmentSession` interface `*alignment.Session` satisfies, and the parent-review gate reaches it through `parentreview.Aligner`, so `internal/parentreview` keeps its import set.

### Decision: alignment-record-layout

Alignment records sit under `shedrun.AlignmentDir(l, runID, point)`, `_lyx/shed/<run>/alignment/<point>/attempt-<N>/`, beside the arbiter's.
Each holds `attempt.json`, `prompt.md`, and the token-named `verdict-<token>.yaml` and `report-<token>.md`, and the status commit stages the `alignment` directory.
The parent-review side passes the loop's run id; the PR side passes `shedrun.SelfRunID`, as `landingDeps`' status checkpoint already does.

### Decision: alignment-idempotence-key

`Session.Begin` reuses any attempt with the same key, which is `round-<N>` at parent review and `pr-<number>-<head>` at PR-Gate.
A driver restart therefore never spawns a second agent for a round or a head.
A closed attempt is never polled for its verdict again, so a verdict written after a fallback, a parent verdict, a pause or a goto is dropped.

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
