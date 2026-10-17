# Card 1 — decision-reason-once

**Create:**
- `plan:internal/alignment#DecisionRecordedReason` -> `const DecisionRecordedReason = "a decision was recorded for this task while the alignment agent ran"`

**Delete:**
- `internal/landingshed#decisionRecordedReason`
- `internal/loomcli#decisionRecordedReason`

**Edit:**
- `internal/landingshed#PRAlign.apply`
- `internal/loomcli#parentReviewAligner.Poll`
- `internal/landingshed#TestPRAlign`
- `internal/loomcli#TestParentReviewAligner`

**Intent:** D4: the reason an approve becomes an escalation when the decision snapshot moved is declared once, as the exported constant `DecisionRecordedReason` in `internal/alignment`, next to `Snapshot` in `records.go`, with a doc comment saying it is the reason both the PR-Gate point (`internal/landingshed`) and the parent-review point (`internal/loomcli`) give.
Its text stays exactly "a decision was recorded for this task while the alignment agent ran".
Remove `decisionRecordedReason` from `internal/landingshed/pralign.go`'s reason const block, keeping the block's other two reasons, and remove the const and its comment from `internal/loomcli/alignmentwiring.go`.
`PRAlign.apply` and `parentReviewAligner.Poll` use `alignment.DecisionRecordedReason`; both files already import `internal/alignment`.
The tests that compare against the reason, the `TestPRAlign` row "an approve over a changed snapshot leaves a note and no record" and the two `TestParentReviewAligner` rows with `wantReason: decisionRecordedReason`, compare against `alignment.DecisionRecordedReason` instead; their behaviour is unchanged, so this card is a refactor the existing rows cover.
After the card, `grep -rn "a decision was recorded for this task" --include=*.go internal` finds the text once, at the declaration.

**ImpactSummary:** one reason constant moves into `internal/alignment`; two producers and their two tests retarget to it, and the reason text is unchanged.

**Commit:** 1: Declare the decision-recorded reason once in internal/alignment

**Verify:**
```bash
test "$(grep -rn 'a decision was recorded for this task while the alignment agent ran' --include=*.go internal | wc -l)" -eq 1
```
