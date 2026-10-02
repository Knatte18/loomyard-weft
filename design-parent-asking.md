## Goal

Discussion-Write holds a live dialogue with whoever spawned the run.
It sends its questions to `$LYX_PARENT` (the operator when no parent is recorded) and waits in the same session for the answer.
When its draft is ready it sends that too, and fixes the parent's findings itself before exiting, so the writer keeps its context throughout.
Shuttle is made to support this.

## Decided with the operator (2026-10-01/02)

- The parent answers every question the child asks; where they disagree, they discuss it out in the dialogue rather than either one deciding alone.
- No separate parent-review producer: a reviewing producer has no one to fix its review.
  This replaces the review gate in `orch-discussion-review` below; its live-question half is what this task builds.
- Later, not now: the same for Plan-Write.

## Shape

- Shuttle: `Spec.AwaitOperator` (`internal/shuttleengine/spec.go`) becomes a choice of who answers, operator or parent.
  An asking session stays non-terminal either way (`internal/shuttleengine/wait.go`); with a parent, a bound expires into `awaiting` with the question surfaced for the operator.
- Discussion-Write's stencil: ask through SendMessage to `$LYX_PARENT` and end the turn; the parent answers by SendMessage to the writer's name.
- Verify early and live: SendMessage reaches an idle interactive session in tmux, and the tool is available inside shuttle runs.

## From `orch-discussion-review`: The orchestrator takes part in Discussion: questions to the parent and a review gate

Operator 2026-09-30/10-01: a weakness that the orchestrator is not involved in discussion; the hub must take part in the design.
Two mechanisms, built in this order:
1. Review gate (millhouse mill-start --orch with orch-review/orch-wait): a seed flag (batten sets it for hub runs) makes Discussion-Review round 1 an `awaiting` stop naming the discussion file; the parent is notified (message to $LYX_PARENT, with the hub's state watch as fallback); the hub forks a reviewer that writes the review in the review-output format (BLOCKING/NIT, class design/scope/decision/consistency, plus review-attack-surface's question) and records it with a verb in the approve/reject idiom; `lyx loom start` resumes; convergence needs two consecutive APPROVEs so round 1 alone never converges. File-based, so it survives a hub restart.
2. Live questions: where Discussion-Write would ask the operator interactively (as millhouse does), it sends each question to $LYX_PARENT and waits; the parent answers by message. shuttle already classifies an asking session (OutcomeAsking, AwaitOperator); route that to the parent instead of the operator. Needs process-naming (names and parentage). No parent or no answer within a bound: fall back to awaiting for the operator.
Open: the same for Plan-Review round 1.