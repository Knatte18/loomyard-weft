---
verdict: APPROVED
---

The review range is `0c4b5acdd..HEAD`, one commit, dc368e78a.
It rewords two rendered strings in `internal/driftengine/verdict.go` and updates their two `TestVerdict` expectations.

Each focus entry, settled:

- `renderVerdict` and `scopeText`: the baseline line now reads "the base of this branch", the wording the board entry asks for.
  The off-scope line now reads "this change is not regression-checked".
  "the task" no longer appears in any line the block renders.
  The line that remains is "Amendments by the run's parent", which the board entry does not cover (see Rejected).
- Other renderers: `Verdict` is the block's only renderer.
  Its callers at `internal/loomcli/wiring.go:1100` and `internal/loomcli/landingdeps.go:293` pass the string through.
  No code outside `verdict.go` matches the changed literals.
  The internal texts that still say "the task's branch point" are the drift expect stencil, `drift capture` help and code comments, and agents, operators and developers read them, not PR readers.
- `Root-Cause:` names the defect the diff removes: `renderVerdict` and `scopeText` hard-coded lym's internal task wording into the repository-facing lines.
- `TestVerdict`: both edited expectations still compare the whole rendered block exactly, and each row still checks the case its name gives.
  No assertion was widened or dropped.
  The off-scope branch with no source (`verdict.go:102`) was already uncovered at the review base.
  The diff only rewords it, and the line is read only by people, so it needs no new row.
- `_lym/landing/summary.md`: the diff bears out each claim.
  The two-way door call holds: the diff changes free text that no binary parses, and it changes no record format, config meaning or push.
  Its blast radius, drift, covers what the diff can break.

## Rejected

- code-scope NIT on `internal/driftengine/verdict.go:152`, "Amendments by the run's parent": the board entry's stated expectation is the baseline line, and the summary gives a reasoned choice to keep this line because it names who changed the expectation.
  Changing it is a wording preference beyond the entry, and filing it would force an edit the answer key does not ask for.
