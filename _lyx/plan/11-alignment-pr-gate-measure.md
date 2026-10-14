# Card 11 — alignment-pr-gate-measure

**Create:**
- `contracts/stencils/loom/loom-fragment-pr-gate-loom.md`
- `contracts/stencils/loom/loom-fragment-pr-gate-darn.md`
- `plan:contracts/stencils#LoomFragmentPRGateLoom` -> `var LoomFragmentPRGateLoom []byte`
- `plan:contracts/stencils#LoomFragmentPRGateDarn` -> `var LoomFragmentPRGateDarn []byte`
- `plan:internal/alignment#MeasureLoom` -> `const MeasureLoom = "loom-fragment-pr-gate-loom"`
- `plan:internal/alignment#MeasureDarn` -> `const MeasureDarn = "loom-fragment-pr-gate-darn"`
- `plan:internal/alignment#readMeasure` -> `func readMeasure(stencilsDir, name string) (string, error)`

**Edit:**
- `contracts/stencils#entries`
- `contracts/stencils/loom/loom-template-alignment.md`
- `contracts/stencils#wordingClaims`
- `internal/alignment#PromptInput`
- `internal/alignment#Prompt`
- `internal/alignment#TestPrompt`
- `internal/alignment#sessionHarness.spawnInput`
- `internal/alignment/doc.go`
- `internal/loomcli#newParentReviewAligner`
- `internal/loomcli#landingAlignmentDeps`

**Uses:**
- `internal/alignment#valueOrNone`

**Intent:** Take what the alignment agent measures a run against at `pr-gate`, and what counts there as a scope deviation, from a told value, so the stencil names no recipe row and darn's wiring can tell its own rules (D10, D9).

Two fragments, registered in `contracts/stencils/stencils.go` like the webster fragments (an embedded var and an `entries` row each, in the loom block), hold the told text.
`loom-fragment-pr-gate-loom.md`: judge the run against its board entry's goal and the decisions recorded in the decision record; a scope deviation is a change the goal does not call for, a goal point the change leaves out, or a recorded decision the change breaks.
`loom-fragment-pr-gate-darn.md`: measure the diff against the board entry as amended by the decision record's `Added after Discussion` entries, an entry by the parent, the operator or the arbiter winning over the board body where they differ; the record's other sections are the writer's own claim, made before coding and reviewed by no one, checked against the board entry and never the measure; a scope deviation is a change the amended entry does not call for, or a point of it the change leaves out, and a breach of the writer's own decisions is not one; an open risk carried over from a review segment that names an unrecorded parent correction is escalated to the run's parent, never approved or rejected, since only the parent records or refuses the correction.
Each fragment carries a leading comment saying which wiring tells it and that it fills the alignment stencil's `pr_gate_measure` marker.

In `loom-template-alignment.md`, the `pr-gate` judge paragraph reads the open risks the review segments carried over into the decision record as part of the run, naming no segment, followed by `{{.pr_gate_measure}}`; `Merge-Review` no longer appears in the stencil.
The header comment lists the marker and says `{{.plan_dir}}` reads `none` for a run with no plan.

`PromptInput` gains `MeasureStencil`, the registered name of the fragment, required.
`Prompt` refuses an empty `MeasureStencil` with an error naming the field, reads the fragment through `readMeasure` (`stencilstore.Read`, then `stencil.StripLeadingComment`, trailing newlines trimmed, as webster's `readFragment` does), wraps a read error with the fragment's name, fills it as `pr_gate_measure`, and fills `plan_dir` through `valueOrNone`, so an empty plan directory renders `none` like the brief path, the pull request number and the head.
Its doc and `PromptInput`'s say which fields may be empty.
`MeasureLoom` and `MeasureDarn` name the two fragments.
`newParentReviewAligner` and `landingAlignmentDeps` set `MeasureStencil: alignment.MeasureLoom`, so both points keep loom's measure until card 12 tells darn's.

`TestPrompt`'s base input sets `MeasureLoom`; the loom `pr-gate` row also wants `a recorded decision the change breaks`; a new darn `pr-gate` row with an empty `PlanDir` and `MeasureDarn` wants ``the plan in `none` ``, `Added after Discussion` and `unrecorded parent correction` and, through a new not-contains list on the row, not `a recorded decision the change breaks`; two new subtests refuse an empty `MeasureStencil` and an absent fragment, each naming it.
`sessionHarness.spawnInput`, whose `PromptInput` reaches `Prompt` through `Session.Begin` (`internal/alignment/session.go:204`), sets `MeasureStencil: MeasureLoom`, so `TestSession` keeps composing its prompt.
The darn fragment's escalate rule needs no new Go path: `landingshed.PRAlign` already leaves an escalate verdict to the parent through PR-Gate, which `TestPRAlign`'s "an escalate leaves a note naming the report and the question" row pins recipe-neutrally.
In `wordingClaims`, the `loom-template-alignment.md` row gains a must claim on `{{.pr_gate_measure}}` and on the open risks the review segments carried over, and a mustNot claim on `Merge-Review`; new rows for the two fragments claim `a recorded decision the change breaks` (loom) and `Added after Discussion`, `a claim`, `unrecorded parent correction` and `escalated to the run's parent` (darn), the darn row also barring `a recorded decision the change breaks`.
`internal/alignment/doc.go` says the agent settles a run's parent review and PR-Gate, and that the `pr-gate` measure is a told fragment.

**ImpactSummary:** the alignment prompt gains one required told field and a `none` plan directory; both loomcli wirings set loom's measure, so a loom run's rendered prompt keeps its meaning.

**Commit:** `11: alignment takes its pr-gate measure from a told fragment`
