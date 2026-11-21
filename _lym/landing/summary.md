# Discussion records a bug entry's fault, root cause and fail-on-base test

The red-first gate on the `Darn-Write` row and the `Root-Cause:` trailer check a bug fix after it is written.
This change has Discussion-Write settle the same things before any code:
for a board entry labelled `bug`, the decision record states what the red-first gate later checks.

## What changes

- `contracts/specs/loom-discussion-spec.md` gains a `### A bug entry's record` rule.
  `## Goal` names the observed fault and how to reproduce it.
  `## Constraints` states the root cause with its evidence, or what is still unknown about it.
  An `## Acceptance criteria` bullet names the test that fails on the merge base and passes after the fix, or states why no test seam exists.
- `contracts/stencils/loom/loom-template-discussion.md` has the writer settle all three for a bug entry.
  The fault and cause are facts Step 2 researches, not interview questions,
  and the stencil points at the spec's rule instead of restating the format.
- `contracts/stencils/loom/loom-rubric-discussion-review.md` flags a bug entry's record that omits any of the three.

## Design decisions

- No new record section and no parser change.
  The three items go into the existing required sections,
  so `internal/discussionparser`, the record splicers and the plan writer's reads stay as they are.
  The rule is enforced by the Discussion-Review rubric, not by a mechanical check.
- A record's statement that no test seam exists does not waive the red-first gate.
  The gate still lifts only on the parent's `### Added after Discussion` entry titled `No test seam`, and the spec says so,
  so a Discussion writer cannot waive the check it states.
- The stencil carries no `{{if}}` on the label.
  It has none by design, and the writer reads the labels from the board entry it already reads in Step 1.

## Merge danger

**Door:** two-way
**Blast radius:** discussion
