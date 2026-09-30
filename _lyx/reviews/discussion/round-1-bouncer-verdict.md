---
verdict: APPROVED
rationale: "All four round-1 findings (F1 index-only drop leaves records on disk, F2 adopt-path crash window, F3 single-commit test anchor, F4 import-list fact) are fixed in the current decision record and support log, and a spot-check of the code claims (battenshed create/teardown Stuck on contention, overview.md frozen-copies sentence, the named CONSTRAINTS.md invariants) found no new defect."
---

# Discussion-Review round 1 — judge verdict

The round-1 report raised four findings; the fixer applied all four, and each fix holds against the current pair.

- **F1 (BLOCKING), resolved.**
  Constraints now carries a "No run records on disk" bullet explaining why an index-only removal after `git worktree add -b` fails.
  The Fabric Destruction Chokepoint bullet names the two mechanisms that satisfy it and leaves the choice to Plan, consistent with Question ledger #6 ("Open for Plan").
  Acceptance criterion 1's "has no such files on disk" is now satisfiable by either mechanism.
- **F2 (MEDIUM), resolved.**
  Decision 2's rationale is narrowed to branches that survive `Remove` and `Add`'s rollback.
  The crash window, the manual recovery that reaches the adopt path, and the remedy are an Open risk, and the rejected adopt-path drop sits in the support log.
- **F3 (LOW), resolved.**
  Decision 5 fixes the message to `fabric: record parent branch for <slug>`, and criterion 1 requires the new weft branch to be exactly one commit ahead of the fork point.
- **F4 (NIT), resolved.**
  The `shedrun` import-list sentence is gone from Decision 4; support-log Interview turn 5 keeps the cycle check.
  This was a legitimate exclusion finding under the writer stencil's Step 2 ban on dependency lists, and no requirement was lost.

The report has no `## Focus departures` section, so there is no departure to ratify.
Its `## Focus verdicts` section addresses all four round-1 focus entries, and I agree with each: Decision 7's sleep seam and info log line carry requirements the unit criteria and operators depend on, and the hand-cleared-pairs assumption is the premise behind the Scope's "inert" exclusion.

Spot-checks against the code agree with the record:
`internal/battenshed/create.go` and `teardown.go` return `Stuck` on the first contended `Acquire`;
`docs/overview.md` carries the "frozen copies … inert" sentence the criteria replace;
and the Shed Run-Directory, Fabric Git, Fabric Destruction Chokepoint, Batten Bookend, Mutation Record and Transient Stop invariants all exist in `CONSTRAINTS.md`.

One non-blocking observation for Plan-Review, not a finding:
Decision 1 says the parent's records "never reach the new weft worktree's disk", which is stronger than the Constraints bullet it rests on ("must never sit in the new weft worktree untracked").
If Plan picks the working-tree-delete mechanism, the records do reach disk briefly before the delete.
The Constraints bullet is the operative requirement and names both mechanisms, so the plan's stated choice resolves this without a Discussion change.
