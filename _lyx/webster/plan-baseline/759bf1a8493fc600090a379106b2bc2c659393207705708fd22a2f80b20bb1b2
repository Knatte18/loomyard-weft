---
format: 5
approved: true
language: go
---

# Plan: webster-stability

Close the Webster faults outside the audit that stop or mislead runs: a gated run's waiting turn end no longer counts at once while a shell is outstanding (D1);
the verify gate names Master's way forward in its re-prompt and its findings (D2);
recovery sessions are the run's own for write evidence, and a non-done recovery exits non-zero (D3);
the implementer body says which commands are always allowed and what a tagged test gets (D4);
the done-check findings carry the lookup (D5);
and a member declared once per mutually exclusive build-constraint set resolves as one member in every planglyph status policy (D6).
The cards run shuttleengine first, then websterengine and webstercli, then the stencil, then planglyph: the classifier lands as a pure addition before any policy reads it.

## Card Index

1 — gated-turn-end-holds — a gated run's waiting turn end never counts at once; drop the rule and its dead fields
2 — shell-result-unread-wording — webster's shell warning, note and summary section say the shell's result was never read
3 — gate-entry-reprompt — a gate entry may supply its own one-line re-prompt renderer
4 — verify-gate-way-forward — the verify entry's re-prompt and findings open with Master's fixer-fork way forward
5 — recovery-sessions-are-run-writes — record every recovery session on the batch and count its writes as the run's own
6 — contract-evidence-names-the-writer — contract-file evidence counts recovery writes and names a fork or a recovery session
7 — recover-batch-non-done-exits-non-zero — a terminal stuck or dead recovery exits non-zero with the digest fields
8 — implementer-allowed-commands — the implementer body names the always-allowed commands and the tagged-test rule
9 — build-constraint-partition-classifier — classify an ambiguous answer as partitioned over the enumerated build environments
10 — status-policies-read-the-classifier — statusFindings and createFindings read the classifier and render constraints
11 — done-check-details-carry-the-lookup — done-check details name the lookup; a partitioned Create or Rename-new lands
12 — answer-symbols-read-the-classifier — delete-order, plan-gate and caller-coverage passes see a partitioned member's declarations
13 — rename-old-side-partitioned — a partitioned Rename old side names its candidates and the way forward

## Shared Decisions

### Decision: classifier-takes-the-worktree-root

- **Decision:** each planglyph pass classifies each `ambiguous` answer it reads once, through `classifyAmbiguity(worktreeRoot, candidates)`.
  A policy that visits each answer once (`statusFindings`, `createFindings`, `doneCheckVerdicts`, `renameDeclSource`) calls the classifier itself and gains a `worktreeRoot` parameter;
  a pass that consults one answer many times (`LaterDeleteReferences`, `planGatePass` and the caller-coverage walk it runs) builds a `partitioned` map once through `partitionedAnswers` and threads the map rather than the root.
- **Rationale:** quarry's `ResolveResult` cannot carry the classification, and threading the told root or the per-pass map keeps the one-batched-Resolve discipline and the status tripwire's allowlist unchanged, while each candidate file is read once per pass, as D6 bounds it.
- **Applies to:** cards 10 to 13

### Decision: wording-cards-grep-the-old-text

- **Decision:** a card that rewords a user-visible message greps the repository for the old text and updates every test asserting it under that card's own gate.
- **Rationale:** a stale exact-string assertion then fails on the card that reworded it, not at the plan-level gate.
- **Applies to:** cards 2, 6 and 7

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
