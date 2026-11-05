# The arbiter settles decision escalations and records its own calls

Review escalations reached the orch for run-level calls the arbiter could have made: whether a fix in the record holds, whether to accept at the hard cap, small rulings inside the board entry's design.
Two limits sent them on: a `decision` escalation was not eligible for the arbiter, and an eligible arbiter could only resume or hand on, never record the ruling it had found.

## What changes

- **Eligibility.** `arbiterstop.Eligibility` admits an awaiting stop with a pending escalation of cause `decision`, as it admits `hard-cap` and `circling`.
  A `budget` escalation, a decision halt and a landing halt still go to the run's parent.
  The driver's escalation fact, `lym loom arbiter` and batten's eligible-stop hold all answer through this one rule, so all three follow.
- **Arbiter stencil** (`contracts/stencils/loom/loom-template-arbiter.md`).
  When the run's repair verbs include `lym loom decision add`, the arbiter settles every call the board entry's design and the decision record's decisions already settle, as the parent would:
  - it records each call with `--by arbiter`;
  - the one edit its stop asks for, such as the correction of a false clause in the decision record, is a recorded decision that names the clause and its exact replacement;
  - anything wider, such as a code change, is recorded as a decision for the run's own agents to carry out.

  It hands on only a question the board entry and the record leave open, after it has recorded every call of the stop it could settle.
  The per-cause rules gain a `decision` rule, and `hard-cap` and `circling` gain the record-then-accept (or continue) path.
  The stencil also says that the escalation brief's own steps are the parent's procedure, which counts only operator decisions, and that the arbiter's prompt wins where they differ.
- **Outcome notice.** When the arbiter resumes a run at an awaiting stop, the shed driver sends the parent one line naming the run, the row and the verdict file, ending "Nothing to answer."
  It goes through the hand-back's `Notify` seam, only when an orch strand is recorded, and parks nothing.
  Other arbiter and tackler resumes stay silent, as before.
- **Orch role.** `contracts/orchrole/orch-role.md` says that a settled escalation reaches the orch as that one line and needs no action, and that a `decision` escalation reaches it only as the arbiter's hand-on.

## Design decisions

- `--by arbiter` already existed as a decision author, restricted to the run's own arbiter strand and shown as `arbiter` in the entry's heading, so no new author was added.
  The bouncer escalation brief still refuses to count `--by arbiter` entries as operator decisions, which keeps the parent's own rule unchanged when the arbiter hands on.
- The arbiter edits no file but its verdict.
  The decision record is append-only, and `lym loom decision add` refuses a record whose working copy differs from its committed one.
  So the "one edit" is a recorded correction entry, which the run's agents follow over the earlier text.
- Where the arbiter may decide alone is drawn in the stencil: a call the board entry's design and the record's decisions settle.
  What to build, a scope change and a choice the board entry leaves open are handed on.

## Tests

- `TestEscalationEligible`: a pending `decision` escalation is now eligible.
- `TestGoDriver_Settle`: a new row checks that an arbiter resume at an awaiting stop sends one notice, relays no parent notice and finishes the run.
