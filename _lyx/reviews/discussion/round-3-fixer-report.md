# Round 3 fixer report

## Changes

- F1, fixed differently: D8's carrier is now the change description, not a board note.
  The Describe row runs in every loom run, reads the whole decision record and lists what to check or run by hand (`contracts/stencils/landing/landing-template-describe.md`), so the check reaches the PR body and the landing commit's message.
  This replaces the proposed board note filed by the discussion writer with `lyx board upsert`, which was not made for two reasons.
  First, that change has the same defect F1 names: the writer's Step 1 has already run, and no gate message has it reread D8, so it has no scheduled step to file the note in.
  Second, `loom-template-discussion.md`'s "What you may write" limits the writer to its two files, with two gate-asked carve-outs, and neither carve-out is a board write.
  The request to `ly:orch` is gone from D8, its Rationale and Q9, and Q9 says why the board-note and message routes were rejected.
- F2, fixed: D8 now says what the carrier can lose.
  The description reaches an actor only at PR-Gate.
  An orch review of the PR files the board note under the orch's board policy, but an alignment-approved run keeps the check only in the landing commit's message.
  The carrier needs no parent, so a run with no recorded parent carries it the same way.
  D8 also gives the note's trigger (the first Webster runs on a binary deployed from the landing commit), its kind `note`, its label `audit` (matching `suite-weight-quiet-rerun`) and the task slug.
  Q9 says a board note is passive and reaches an actor at triage, not at landing.
- F3, fixed: D8's Decision and Rationale describe the carrier as it will exist.
  Q9 records the real state: no note is filed now, and one is filed only when the orch reviews the PR.
  The sentence about the round-2 fixer's messaging limit is removed.

## Disputed

None.

## Deferred

None.
