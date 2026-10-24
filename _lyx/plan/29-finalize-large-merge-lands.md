# Card 29 — finalize-large-merge-lands

**Edit:**
- `internal/landingshed#Deps`
- `internal/landingshed#Finalize.mergeParent`
- `internal/landingshed#TestFinalize_MergeJudgement`

**Extends:**
- `internal/landingshed/doc.go`

**Intent:**
D11: the beam recipe's Finalize declares no `on_reroute` and must never reroute, since shed fails a run whose row with no `on_reroute` returns a reroute.
`Deps.LargeMergeLands bool`: when set, a merge that needed the conflict session or whose parent changes overlap the task's files is not rerouted and writes no overlap note; it goes on to the `finalize` gate like any other merge.
A spent or over-cap fixer then halts for the orch through the existing path, since the beam's Finalize is built with no `RerouteReviewed` (nil reads as no review).
Bound: what this lets through, unseen by any member's review, is the conflict session's resolution of the beam against its parent's newer work and a textually clean merge of the parent's changes to files the beam also changed; the `finalize` gate over the merged tree bounds both, and a fixer change past the caps, or a spent fixer, halts.
`TestFinalize_MergeJudgement` gains rows with `LargeMergeLands`: a conflicted merge and an overlapping merge each proceed to the gate with no reroute and no overlap note written, while the same rows without the flag still reroute.

**ImpactSummary:** One opt-in flag on Finalize's deps; loom and darn leave it false and keep their reroutes.
