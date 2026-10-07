# Card 8 — set the cautious profile from the calibration report

**Edit:**
- `internal/batcher/template.yaml`

**Uses:**
- `tools/tokencount/calibrate.go`

**Intent:**
Run the calibration report once on the operator's transcripts and, when the fit is good, rescale the `cautious` profile's context-size numbers by one factor (Auto-mode assumptions).
`active` stays on identity.

Hub edit after batch 08 failed three times on the fabric-reference audit: do not run the calibration report in this card.
It needs the hub's plan-artifact history, which lives only in the fabric repository, and an implementer fork must never read or name that repository or any sibling checkout outside this worktree.
The hub runs the calibration from its prime after this task lands; an earlier attempt measured 165 cards over 14 runs with a median ratio of 0.044 and a spread of 4.45, a poor fit.

This card therefore always takes the "report cannot be produced" branch: run no `tokencount` command, change nothing, make no commit, and state in the batch report that the calibration is left to the hub for the reason above.

**ImpactSummary:** nothing changes in this card; `cautious` and `active` keep their template numbers, so no run's batching changes.
