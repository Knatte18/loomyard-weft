## Goal

Port millhouse's `orch-review`/`orch-wait` as a review step in Discussion-Write's own producer gate: when the writer reports done, the gate asks a reviewer (for now always the parent) to review the discussion, and a reject re-prompts the still-live writer with the findings, so the writer fixes with its own context.
No new recipe row and no separate fixer (operator, 2026-10-02).
The interview helper is the separate note `discussion-advisor`.

## Shape

- Discussion-Write's gate today is the mechanical `discussion` validator (`internal/shedrecipe/entries_gate.go`, `internal/shuttleengine/gate.go`).
  A gate is Go code run before the producer may finish; it returns pass or fail, and what it checks is up to the program it calls.
- One mechanism, one new flag (operator, 2026-10-02): `GateSpec` gains `PassOnCap`.
  With it, a gate that has used its attempts lets the run through on its next arrival instead of failing into Stuck.
  Millhouse's `orch-review` behaviour is `attempts: 1`, `pass_on_cap: true`: the review runs once whatever it finds, the writer addresses it, and the next arrival passes; independent reviews follow in the Burler segment as today.
  `attempts: 0` turns the review off.
- A row can carry more than one gate, each with its own attempts and flag, evaluated in order at every arrival.
  Discussion-Write runs the validator first (must always pass, no `pass_on_cap`), then the review; the validator therefore re-checks the writer's fixes after the review.
- The gate is mechanical in the sense its name means: the LLM inside the producer cannot decide it is done; the gate decides.
  A review by another agent fits that, so no invariant changes; the spec and CONSTRAINTS wording should say so plainly.
- The reviewer is an address, defaulting to `$LYX_PARENT`, so others can review later; only the parent is wired now.
- A lyx verb pair carries the exchange (names settled in Discussion):
  - the gate runs one to open a review request: write the request file, notify the reviewer by name (agent-naming), and wait for the verdict file;
  - the reviewer runs one to submit an approve or a reject with its review file.
  The parent reviews through a one-shot fork that writes the review in the review-output format (BLOCKING/NIT, class design/scope/decision/consistency), so its own context grows by a notification and a summary per round.
- Waiting on a reviewer takes minutes, not milliseconds:
  - the run's timeout must allow it;
  - status must show that the run waits on its reviewer (and who), not a plain `running`;
  - request and verdict are files, so a driver restart that re-attaches (`AttachGated`) finds the open request and keeps waiting rather than asking twice.
- A writer that disagrees with a finding answers it in the discussion file instead of fixing it; the next round's reviewer reads that answer.
- A run with no parent recorded, or `attempts: 0`, skips the review step.

## Bolt as design context (operator, 2026-10-02)

Bolt exists to show what is planned ahead, so both Discussion-Write and the reviewer read it: the design must fit planned work, and must not duplicate or contradict it.
Discussion-Write's stencil tells it to read `lyx board list` and `lyx board notes list` (and `get` what touches its task); the reviewer's brief does the same.

## From `orch-discussion-review`: The orchestrator takes part in Discussion: questions to the parent and a review gate

Operator 2026-09-30/10-01: a weakness that the orchestrator is not involved in discussion; the hub must take part in the design.
Two mechanisms, built in this order:
1. Review gate (millhouse mill-start --orch with orch-review/orch-wait): a seed flag (batten sets it for hub runs) makes Discussion-Review round 1 an `awaiting` stop naming the discussion file; the parent is notified (message to $LYX_PARENT, with the hub's state watch as fallback); the hub forks a reviewer that writes the review in the review-output format (BLOCKING/NIT, class design/scope/decision/consistency, plus review-attack-surface's question) and records it with a verb in the approve/reject idiom; `lyx loom start` resumes; convergence needs two consecutive APPROVEs so round 1 alone never converges. File-based, so it survives a hub restart.
2. Live questions: where Discussion-Write would ask the operator interactively (as millhouse does), it sends each question to $LYX_PARENT and waits; the parent answers by message. shuttle already classifies an asking session (OutcomeAsking, AwaitOperator); route that to the parent instead of the operator. Needs process-naming (names and parentage). No parent or no answer within a bound: fall back to awaiting for the operator.
Open: the same for Plan-Review round 1.