# Card 5 — A relaunched Discussion writer finishes all three files

**Edit:**
- `internal/twilldirective#Values`
- `internal/twilldirective#Directive`
- `internal/twilldirective/doc.go`
- `internal/twilldirective#filledValues`
- `internal/twilldirective#TestDirective_RendersEveryValueVerbatim`
- `internal/twilldirective#TestDirective_MissingValueNamesTheMarker`
- `contracts/stencils/twill/twill-relaunch.md`
- `contracts/stencils#TestStencils_TopLevelMarkers`
- `internal/loomengine#discussionLensDirective`
- `internal/loomengine#TestDiscussionTable`

**Uses:**
- `internal/loomengine#DiscussionRelevant`
- `internal/loomengine#DiscussionDecisionRecord`
- `internal/loomengine#DiscussionSupportLog`

**Intent:**
Per D1, a relaunched Discussion writer finishes `relevant.md` too, while the lens forks still review only the decision record and the support log.

`twilldirective.Values` gains `RelaunchFiles string`, documented as the phrase naming the files a relaunched writer finishes, rendered verbatim with the host's own backticks, and read only when `WriterFork` is nil.
`Directive` fills the relaunch part from `mergedValues(values, map[string]string{"relaunch_files": v.RelaunchFiles})`, so the marker is required only where `twill-relaunch.md` is rendered; the writer-fork branch never reads it, and the Plan host, which sets `WriterFork`, supplies no value.
`doc.go`'s list of host-specific values gains the files a relaunched writer finishes, for a host without a writer fork.

`twill-relaunch.md`'s sentence "It then finishes writing the reviewed files per ..." names `{{.relaunch_files}}` in place of "the reviewed files"; the read-back sentence keeps "every reviewed file", so the forks still inherit the reviewed files.
Its leading comment adds `{{.relaunch_files}}` to the required markers it declares, and the stencil stays host-neutral.
`TestStencils_TopLevelMarkers`'s `twill-relaunch` row adds `relaunch_files`.

`discussionLensDirective` sets `RelaunchFiles` to the three backticked paths, `DiscussionDecisionRecord(layout)`, `DiscussionSupportLog(layout)` and `DiscussionRelevant(layout)`, and its `SpawnPoint` says "Once you have written all three output files per Step 5" in place of "both output files".
`ReviewedFiles` is unchanged.

Tests:
- `filledValues` sets `RelaunchFiles: "RELAUNCH-FILES"`.
- `TestDirective_RendersEveryValueVerbatim`'s "without a writer fork" row wants `RELAUNCH-FILES`; its "with a writer fork" row sets `RelaunchFiles` to empty and still renders without error, and lists `RELAUNCH-FILES` among its `wantNots`.
- `TestDirective_MissingValueNamesTheMarker` gains the row `RelaunchFiles without a writer fork`, emptying `RelaunchFiles` and wanting the error to name `relaunch_files`.
- `TestDiscussionTable`'s twill subtest wants `finishes writing ` immediately followed by the rendered `RelaunchFiles` phrase, the three backticked paths in the form `discussionLensDirective` builds, in the lead prompt, a string only the relaunch sentence produces, so it fails if the value is dropped even once card 6 names the relevant path elsewhere in the prompt.

**ImpactSummary:** one new `Values` field required only on the relaunch branch; the only host without a writer fork is `discussionLensDirective`, on this card, and the Plan host is untouched.

**Commit:** 5: twill: a relaunched Discussion writer finishes the relevant list too
