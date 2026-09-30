---
format: 5
approved: false
language: go
---

# Plan: Publish review fix-back

Give loom a path back from a reviewed pull request into the run.
Publish splits into Publish (sync, verify, push, open or refresh the PR) and a new `PR-Gate` row that owns approval, rejection and the wait.
A new `lyx loom reject <review-file>` verb records the operator's findings, the gate bounces to a new `PR-Rework` row that appends cards to the plan, re-baselines Webster's plan fingerprint and routes to `Webster`, and the main line re-runs through review, describe, publish and the gate.
Batten resumes an awaiting child on either operator decision.

## Card Index

1 — landingshed-rejection-record — add the Rejection record, its reader and writer, and decision-record removal
2 — loomengine-rework-paths — add the rejection, coverage and rework-round path accessors
3 — planparser-parse-from — parse a plan through a file-reader closure
4 — planparser-append-only — compare a base and an extended plan for append-only changes
5 — fabricengine-committed-file — read an anchored file as committed at HEAD
6 — websterengine-rebaseline — export the plan-fingerprint re-baseline and prove resume over an extended plan
7 — landingshed-pr-gate — add the PR-Gate producer owning every review decision
8 — rework-stencil-spec — add the rework stencil and its Spec factory
9 — loomshed-pr-rework — add the PR-Rework producer with its append-only check and round record
10 — shedrecipe-review-entries — register the PRGate and PRRework engines
11 — loom-recipe-review-rows — route Publish through PR-Gate and PR-Rework and wire the rework seams
12 — landingshed-publish-split — Publish returns Done at an open PR and refreshes its title and body
13 — loomcli-approve-at-gate — approve at the gate row, removing a pending rejection
14 — loomcli-reject-verb — add lyx loom reject with its refusal table
15 — batten-decision-resume — batten resumes an awaiting child on an approval or a rejection
16 — pr-fixback-docs — document the gate, reject and the hand-made-fix rule

## Shared Decisions

### Decision: names

- **Decision:** the gate row is `PR-Gate` (engine `PRGate`), the rework row is `PR-Rework` (engine `PRRework`), and their segment is `PR-Review` — the decision record's working names, kept.
- **Applies to:** cards 7, 9, 10, 11, 13, 14, 16

### Decision: rework reuses the plan role

- **Decision:** the rework session takes the plan role's model-spec and `plan_timeout_min`; no `loom.yaml` key is added.
- **Rationale:** rework is planning, and a new key would force a config reconcile on every hub for no tunable anyone asked for.
- **Applies to:** card 8

### Decision: round record layout

- **Decision:** each rejection round lands as `_lyx/loom/rework/round-<N>/` holding `findings.md`, `record.json` (`pr_number`, `head_sha`, `rejected_at`) and `coverage.md` (the agent's finding-to-card map, its completion file), committed with the appended cards.
- **Applies to:** cards 2, 8, 9, 11

## verify:

go build ./...
go vet ./...
go test ./...
go test -tags integration ./...
go vet -tags smoke ./...
