---
format: 5
approved: false
language: go
---

# Plan: webster-replan-state

Webster stays consistent across replans, its batch and fabric checks judge only what they should, implementer forks stay sandboxed, the shell wait leaves nothing unrecorded, and the round gate re-runs whole tests.
Each card lands one decision of the record with its test, its doc and its refusal-spec row in the same commit.

## Card Index

1 — gitexec-run-stdin — a checked gitexec form that feeds git its stdin
2 — commits-named-by-one-batch-check — classify every disambiguated object in one cat-file batch-check
3 — rebaseline-refusals-keyed — key every rebaseline refusal on the caller's step and name the empty-frame cause
4 — run-auto-rebaseline — the Webster row rebaselines a changed plan on entry behind a run option
5 — webster-row-auto-rebaseline-stuck — the shed adapter sets the option and maps its errors to Stuck
6 — edited-done-cards — which done cards were edited since their batch began
7 — plan-gate-done-card-edited — the plan gate refuses an edit to a done card
8 — webster-record-note — render webster's run record for the Plan-Review rubric
9 — plan-review-webster-record-marker — the rubric's webster_record marker and its plumbing
10 — verify-chain-splitter — one chain splitter for the two verify checks, splitting at || and |
11 — pattern-entry-line-cap — fail a card prescribing a PATTERN entry line over the cap
12 — done-checks-name-the-batch — a done-check refusal opens with the batch and its cards
13 — read-only-classifier-allow-list — the read-only classifier admits plain-word lyx help, fabric readers and git reads
14 — fabric-reference-policy-at-record-batch — a read-only fabric reference is a policy warning
15 — recover-batch-accepts-read-only-fabric-references — recover-batch accepts them in line, with an envelope flag and a stencil rung
16 — fabric-sandboxed-role-guard — lyx fabric refuses its mutating verbs from a sandboxed role
17 — gate-and-verify-drop-the-strand-name — gate and verify runs hand their tests no strand name
18 — implementer-sandbox-rule — the implementer stencil states the sandbox rule
19 — fabric-way-forward-names-the-orch — every refused fabric verb's way forward names the orch
20 — turn-end-counts-with-nothing-outstanding — webster's turn-end reading drops the shell expiry
21 — one-shell-wait-rule-and-ended-shells — shuttle waits on both shell signals alike and records every shell a run ends over
22 — round-gate-reruns-the-top-level-test — the Publish failure re-run names the top-level test only
23 — recovery-count-and-retry — count recovery spawns, retry a dead recovery that committed, cap at two
24 — recovery-retry-on-the-verbs — the verbs carry the count and the retry, and Master runs the second recovery
25 — resume-reconcile-determinism — reproduce and remove the resume-reconcile test's nondeterminism

## Shared Decisions

### Decision: refusal-and-doc-in-the-same-commit

- **Decision:** A card that adds or rewords a refusal in webster, shed, loom or gate carries its `contracts/specs/refusal-spec.md` row, and a card that changes a module's observable behaviour carries that module's `doc.go` passage.
- **Rationale:** PATTERN-refusal-way-forward and PATTERN-documentation-lifecycle bind every commit, not the plan as a whole.
- **Applies to:** all cards

### Decision: classifier-by-injection

- **Decision:** `websterengine` keeps importing no `fabricengine`: the read-only classifier reaches `ClassifyViolation`, `RecoverDeps` and `AcceptBatchFabricReference` as a `func(cmd string) bool`, and `webstercli` supplies `fabricengine.IsReadOnlyCommand`.
- **Rationale:** the engine/cli split in `internal/websterengine`'s package doc.
- **Applies to:** cards 13, 14, 15

### Decision: step-keyed-refusal-texts

- **Decision:** A rebaseline refusal names the step its caller passes, `lyx webster rebaseline` by default and the Webster row's re-step under the loom, so no auto-path text tells an agent to run a verb by hand.
- **Rationale:** PATTERN-no-denied-recovery and D1's bound that the auto path accepts exactly what `--card` accepts.
- **Applies to:** cards 3, 4, 5

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
