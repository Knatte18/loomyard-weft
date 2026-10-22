# Card 9 — darn-write-reroute-edge

**Edit:**
- `contracts/recipes/darn-recipe.yaml`
- `internal/loomrecipe#TestNewDarn_Routing`
- `internal/loomcli#latestDarnOutcome`
- `internal/loomcli#TestLatestDarnOutcome`

**Intent:** D3's routing half: the darn recipe gives `Darn-Write` an `on_reroute: Merge-In` edge, and the prior-work note's reader of the latest Darn-Write outcome skips a reroute entry, so a later writer spawn names the last real halt.
Card 10 makes the producer return the reroute; until then the edge is unused.

`darn-recipe.yaml`'s Darn-Write row gains `on_reroute: Merge-In` with a comment: a Darn-Write call that finds this run's failure record, an unmoved task branch, a clean worktree and a moved parent reroutes to Merge-In before spawning the writer.
The header comment's paragraph on Darn-Write and Describe carrying no on_stuck says Darn-Write's one reroute edge sends a moved parent to Merge-In, the row that merges it, and the paragraph on `on_reroute: Merge-In` names Darn-Write beside Publish, PR-Gate and Finalize.
The edge goes backward past the Darn-Review perch and Describe; it is bounded by card 10's conditions and by shed's reroute cap, which halts the row after its third reroute since its latest Done or cap halt.

`TestNewDarn_Routing`'s first subtest asserts `rows[loomshed.NameDarnWrite].OnReroute` is `loomshed.NameMergeIn`; the loop over Publish, PR-Gate and Finalize takes Darn-Write as a fourth row.

`latestDarnOutcome`, walking the history newest first, skips a Darn-Write entry whose outcome is `shedengine.OutcomeReroute` and takes the next one.
`TestLatestDarnOutcome` gains a row whose history holds a Darn-Write Stuck and then a Darn-Write reroute entry, halted at Darn-Write with an error: the answer is the Stuck entry with the status error as its reason; and a row whose only Darn-Write entry is a reroute, with the run at Merge-In, which reports not found.

**ImpactSummary:** one recipe edge pinned by the darn routing test and one history filter in `latestDarnOutcome`, read only by Darn-Write's prior-work note; loom's recipe is untouched.
