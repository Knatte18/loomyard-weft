## Goal

Port millhouse's `orch-review`/`orch-wait` as one Shed producer, `Discussion-Parent-Review`: the session that spawned the run reviews the discussion once Discussion-Write has drafted it, and the writer fixes the review with its own context.
Scope is this producer only (operator, 2026-10-02); the interview helper is the separate note `discussion-advisor`.

## Shape

- Discussion-Write works as today: it answers its own questions as best it can and finishes the draft.
- `Discussion-Parent-Review` sits right after Discussion-Write and before the Burler segment, and must not assume its position, so a second row after the Burler segment can be added later by recipe alone.
- The row stops as `awaiting` and notifies `$LYX_PARENT` (message to the parent's name; the parent's own state watch is the fallback).
  A run with no parent recorded skips the row.
- The parent forks one one-shot reviewer, which writes the review in the review-output format (BLOCKING/NIT, class design/scope/decision/consistency); the parent records it with a lyx verb in the approve/reject idiom, and `lyx loom start` resumes.
  No fork waits; the parent's cost per round is a notification and the fork's summary.
- On reject, the producer resumes the writer's own Claude session (shuttle's `ResumeCmd` and the recorded `SessionID`, `internal/shuttleengine/engine.go`) with the review, so the fix keeps the writer's context; the writer may answer a finding it disagrees with in the file instead of fixing it.
- Rounds repeat until the parent approves or a round cap is reached; disagreement left at the cap becomes `awaiting` for the operator.
- File-based throughout, so it survives a hub restart.

## Bolt as design context (operator, 2026-10-02)

Bolt exists to show what is planned ahead, so both Discussion-Write and the parent's reviewer read it: the design must fit planned work, and must not duplicate or contradict it.
Discussion-Write's stencil tells it to read `lyx board list` and `lyx board notes list` (and `get` what touches its task); the reviewer's brief does the same.

## Verify early and live

- Resuming a finished writer session under shuttle restores its context and accepts the review as its next turn.

## From `orch-discussion-review`: The orchestrator takes part in Discussion: questions to the parent and a review gate

Operator 2026-09-30/10-01: a weakness that the orchestrator is not involved in discussion; the hub must take part in the design.
Two mechanisms, built in this order:
1. Review gate (millhouse mill-start --orch with orch-review/orch-wait): a seed flag (batten sets it for hub runs) makes Discussion-Review round 1 an `awaiting` stop naming the discussion file; the parent is notified (message to $LYX_PARENT, with the hub's state watch as fallback); the hub forks a reviewer that writes the review in the review-output format (BLOCKING/NIT, class design/scope/decision/consistency, plus review-attack-surface's question) and records it with a verb in the approve/reject idiom; `lyx loom start` resumes; convergence needs two consecutive APPROVEs so round 1 alone never converges. File-based, so it survives a hub restart.
2. Live questions: where Discussion-Write would ask the operator interactively (as millhouse does), it sends each question to $LYX_PARENT and waits; the parent answers by message. shuttle already classifies an asking session (OutcomeAsking, AwaitOperator); route that to the parent instead of the operator. Needs process-naming (names and parentage). No parent or no answer within a bound: fall back to awaiting for the operator.
Open: the same for Plan-Review round 1.