## Goal

The parent (the session that spawned the run; the operator when none is recorded) takes part in Discussion through one of two modes, chosen per run; never both at once.

- **ask**: a running conversation.
  Discussion-Write asks its design questions as they arise, and the parent answers them; it is answers to design questions, not a review.
- **review** (millhouse's `orch-review`/`orch-wait`): Discussion-Write answers its own questions as best it can, finishes the draft, and then asks the parent for input; the parent's review comes back and the writer fixes it.

## Decided with the operator (2026-10-02)

- The parent answers every question; where writer and parent disagree, they argue it out in the files rather than either one deciding alone.
- The writer always fixes; there is no separate parent-review producer (a reviewing producer has no one to fix its review).
- Neither side's waiting may cost the parent its context:
  - The writer is a shuttle-run interactive session and waits on a file; shuttle keeps it non-terminal, as it already does for an asking session (`internal/shuttleengine/wait.go`), and a bound expires into `awaiting` with the open question or draft surfaced for the operator.
  - The parent arms a Monitor and, each round, forks a fresh one-shot agent that reads the exchange file, answers or reviews, and exits; no fork ever waits, so none is flagged dead by its master.
    A fork inherits the parent's context, and the exchange file records every answer with its reasoning, so continuity lives in those two, not in the fork.
  - A lyx verb delivers the parent's answer or review and wakes the writer.
- Rounds repeat until the parent approves or a round cap is reached; disagreement left at the cap becomes `awaiting` for the operator.
- The mode is a seed field set by the spawner (batten for hub runs); no parent means the operator, as today.
  Model it as two independent switches (`ask`, `review`) rather than one enum: one is on per run for now, but the operator sees both on later, served by two different agents, one answering the writer's interview questions and another doing the review (two agents designing together beat one agent asking and answering itself).
- Build the shared waiting first, then `review` (proven in millhouse), then `ask` on the same machinery.
- Later, not now: the same for Plan-Write.

## Verify early and live

- An idle interactive session in tmux is reliably woken by the delivering verb.
- The parent's Monitor-plus-fork loop costs little context per round (measure it on the first hub run).

## From `orch-discussion-review`: The orchestrator takes part in Discussion: questions to the parent and a review gate

Operator 2026-09-30/10-01: a weakness that the orchestrator is not involved in discussion; the hub must take part in the design.
Two mechanisms, built in this order:
1. Review gate (millhouse mill-start --orch with orch-review/orch-wait): a seed flag (batten sets it for hub runs) makes Discussion-Review round 1 an `awaiting` stop naming the discussion file; the parent is notified (message to $LYX_PARENT, with the hub's state watch as fallback); the hub forks a reviewer that writes the review in the review-output format (BLOCKING/NIT, class design/scope/decision/consistency, plus review-attack-surface's question) and records it with a verb in the approve/reject idiom; `lyx loom start` resumes; convergence needs two consecutive APPROVEs so round 1 alone never converges. File-based, so it survives a hub restart.
2. Live questions: where Discussion-Write would ask the operator interactively (as millhouse does), it sends each question to $LYX_PARENT and waits; the parent answers by message. shuttle already classifies an asking session (OutcomeAsking, AwaitOperator); route that to the parent instead of the operator. Needs process-naming (names and parentage). No parent or no answer within a bound: fall back to awaiting for the operator.
Open: the same for Plan-Review round 1.
