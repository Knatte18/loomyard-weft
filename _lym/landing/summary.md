# Discussion records a bug entry's fault, root cause and fail-on-base test

For an entry labelled `bug`, the Discussion-Write lead now records three facts in the decision record:

- the observed fault and the steps that reproduce it, under `## Goal`;
- the root cause and the evidence it rests on, or what is still unknown about it, under `## Constraints`;
- under `## Acceptance criteria`, the test that fails on the merge base and passes after the fix, or why no test seam exists.

The record then states up front what the `lym gate red-first` check on the `Darn-Write` row later verifies.
Before this change the stencil had no bug-specific guidance, so a bug record could omit all three.

## Changes

- `contracts/stencils/loom/loom-template-discussion.md`: when the entry is labelled `bug`, the design settles the three facts.
  The fault and its cause are researched in Step 2, never asked of the operator as interview questions.
- `contracts/specs/loom-discussion-spec.md`: a new "A bug entry's record" section pins where each fact goes.
  The test must reach the bug through code the merge base already has.
  A record's claim that no test seam exists does not lift the red-first gate;
  only the parent's `### Added after Discussion` entry whose title opens `No test seam` does.
- `contracts/stencils/loom/loom-rubric-discussion-review.md`: a bug entry's record that omits the fault, the cause or the test is a finding.

## Design decisions

- The rule applies to the Discussion-Write lead's records only, and is keyed on the board label `bug`.
- The no-test-seam escape stays in the parent's hands, so a writer cannot waive the red-first gate by assertion.

## To check by hand

- The bug-entry rule in `contracts/specs/loom-discussion-spec.md` is scoped to the Discussion-Write lead's records.
  A review flagged it as unscoped; confirm the wording now matches the intent.
- No prose change was run against a live Discussion for a `bug` entry.

## Merge danger

**Door:** two-way
**Blast radius:** stencils
