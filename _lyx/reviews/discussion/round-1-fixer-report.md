# Fixer report — discussion round 1

Every finding's premise was checked against the code: `discussionparser.AppendDecision` checks section presence only, `commitDiscussion` commits through `fabricengine.CommitAnchoredPaths` under the weft write lock and treats an empty commit as no commit, `validate-discussion` commits nothing today, the alignment stencil's `pr-gate` judge text names `Merge-Review` and judges against the record's decisions, and `internal/loomcli/decision.go` holds the caller rule F3 names.
`lyx loom validate-discussion` passes on the edited record.

## Changes

- F1 (fixed as proposed, plus one guard): D4 commits the two files at the writer's first clean `validate-discussion` on a darn run, keeps the handoff gate as the check, and has the gate and a later clean `validate-discussion` refuse a working copy that drops a committed `Added after Discussion` entry.
  The added guard on later `validate-discussion` runs closes the case where a writer overwrite followed by a clean validation would otherwise commit the dropped entry's deletion before the gate could see it.
  The gate's pass still commits, so a writer that skipped the verb still gets a committed record.
  D5 refuses a record not yet committed or differing from its committed copy, with the way forward to message Darn-Write, and its Bound now rests on that check.
  Scope line, two new acceptance criteria and two reworded ones, a Notes bullet on `validate-discussion` learning the recipe, the support log's rejected alternative reworded, and Q10 updated.
- F2 (fixed as proposed): D9 states the darn `pr-gate` measuring rule as a told prompt value and adds it to the Bound sentence; D10 now takes the measuring stick from a told value for both recipes; acceptance criterion added.
- F3 (fixed as proposed): D7 says a named unrecorded parent correction is left open, never reverted, and carried by D8 into `## Open risks`, and names the caller rule as the bound on overriding entries with `--by` unverified; the D7 criterion carries the rubric and `fasit` wording, and a carry-over criterion is added.
- F4 (fixed as proposed): D4's verify-gate ordering clause is gone; D5's collision sentence is replaced by the weft write lock's serialization.
- F5 (fixed as proposed): test-budget and `CLAUDE.md` production-paragraph lines deleted; the docs list moved to Notes, leaving "docs land in the same commit" in Constraints.
- F6 (fixed as proposed): D10 acceptance criterion added.

## Disputed

None.

## Deferred

None.
