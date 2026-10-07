# Round 1 fixer report

All eleven findings are fixed in `_lyx/discussion/decision-record.md` and `_lyx/discussion/support-log.md`.
`lyx loom validate-discussion` exits 0 on the result.

## Changes

- **F1** (folded-in fabric-reference scope):
  - Added D13 (the `card-fabric-reference` Plan-Gate check, with its scan scope and bound), D14 (`accept-audit --batch` for a committed batch whose fabric references are read-only, with a closed reader list and its bound) and D15 (the committed-data steer in the plan template).
  - Updated Goal, Scope In and Out (the audit left Out, replaced by the audit's classification and the imported issues' other options), Constraints, Auto-mode assumptions, Open risks, Acceptance criteria and Notes.
  - Support log: interview step, four rejected alternatives, ledger Q11 to Q13.
- **F2**: D4 states that `budget` bounds only multi-card segments, that an over-budget lone card still runs, and where the operator sees it; an Acceptance criterion and an Open risk were added.
- **F3**: D9 defines the residual spread, the not-fitted case (fewer than two distinct positions), the unusable negative coefficient, and the transcript-based Merriam session; an Acceptance criterion was added.
- **F4**: D5 states both directions of a wrong position and that only the budget headroom bounds the low one.
- **F5**: D11 attributes profile, failed batches and recoveries per batch, counts cards from the recorded partition, marks a `--fresh` restart by differing run guids, and gives a multi-profile run its own row.
  This differs from the review's suggested majority-profile attribution: a separate row never assigns a run's findings to a profile the run only partly used.
- **F6**: D11 and the Constraints name `burlerengine.ParseReview` as the reader of the round files, drop the `PATTERN-sole-parsers` claim for review files, and say that `prior-generation/` archives are not counted.
- **F7, F8**: D12's rubric clause now names a card-order violation and points at the spec section without a case list; the rationale credits Plan-Gate's two checks and `CheckBatchOrder`.
- **F9**: D7 and the Auto-mode assumptions say twelve forks, one in a resumed session.
- **F10**: the D2 and D3 bullets are gone and the first bullet no longer refers to the support log.
- **F11**: the `sed` note is deleted.

## Deferred

Nothing deferred.
D13 to D15 are auto-picks made without the operator; they are listed in the Auto-mode assumptions so the operator can overrule them.
