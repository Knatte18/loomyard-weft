# Parent review as a step in Discussion-Write's gate

`parent-review` · Planned · feature · running · Describe

## Goal

Port millhouse's `orch-review`/`orch-wait` as a review step in Discussion-Write's own producer gate: when the writer reports done, the gate asks a reviewer (for now always the parent) to review the discussion, and a reject re-prompts the still-live writer with the findings, so the writer fixes with its own context.
No new recipe row and no separate fixer (operator, 2026-10-02).
The interview helper is the separate note `discussion-advisor`.

## Shape

- The review is a second gate on Discussion-Write, after the mechanical `discussion` validator, built on `gate-list`'s gate list: `attempts: 1`, `pass_on_cap: true`.
  That is millhouse's `orch-review` behaviour: the review runs once whatever it finds, the writer addresses it, the validator re-checks the fixes, and the next arrival passes; independent reviews follow in the Burler segment as today.
  `attempts: 0` turns the review off.
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