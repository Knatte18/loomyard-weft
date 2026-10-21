# Card 13 — recipe-bouncer-subject

**Edit:**
- `internal/shedrecipe#Env`
- `internal/shedrecipe#SegmentSettings`
- `internal/shedrecipe#bouncerEntry`
- `internal/shedrecipe#burlerReviewEntry`
- `internal/shedrecipe#TestBurlerReviewEntry_RubricStencil`
- `internal/shedrecipe/doc.go`
- `internal/testkit/envkit#FullEnv`
- `internal/testkit/envkit#nilLegal`
- `contracts/stencils#rubricMarkerAllowlist`
- `contracts/stencils#TestLoomRubrics`
- `contracts/stencils/loom/loom-rubric-discussion-review.md`
- `contracts/stencils/loom/loom-rubric-plan-review.md`
- `contracts/stencils/loom/loom-rubric-webster-review.md`
- `contracts/stencils/loom/loom-rubric-darn-review.md`

**Prosa:**
- `contracts/specs/shed-recipe-spec.md`

**Create:**
- `internal/shedrecipe#TestBouncerEntry_Subject`

**Uses:**
- `internal/shedadapters#Subject`
- `internal/shedadapters#AmendmentNote`
- `internal/shedrecipe#layoutBouncerRound1Report`
- `internal/shedrecipe#judgeSeamShuttle`

**Intent:** The recipe wiring of D5's subject seams and D6's allowance, told per segment (`PATTERN-told-geometry`, `PATTERN-recipe-setting-lookup`).

The Bouncer row takes an optional `subject` config key: `document`, `code` or `merge`; an absent key tells no subject and keeps today's paths.
`Env` gains the seams the key selects, each guarded by `requireSeam` when named: `DocumentSubject func(paths []string) (shedadapters.Subject, bool, error)` (told the row's resolved artifact paths), `CodeSubject func() (shedadapters.Subject, bool, error)`, `MergeSubject func() (shedadapters.Subject, bool, error)`, `MergeInOnly func(since string) (bool, error)` (told with `code`), `WeftHead func() (string, error)` (told as `SettledBase` with `document`) and `RejectReview func() (string, time.Time, bool, error)` (optional, nil for none).
`merge` also sets `FullGeneration`.
`SegmentSettings` gains `AmendMaxBounces int`, which `bouncerEntry` tells as `AmendAllowance`.
A `subject` value outside the three is refused naming the key, the value and the three legal ones.
`burlerReviewEntry` adds `shedadapters.AmendmentNote(runDir)` to the notes it builds the profile's rubric from, so the reviewers read the same note as the seed and the judge.

The embedded recipes name no `subject` yet; card 15 adds the keys together with the loom and darn seams that serve them.
The four amendment-capable rubrics gain an `## Amendment` section rendering `{{.amendment}}`: when it is not `none`, review only the change it names, and report nothing about the settled remainder; the merge rubric gets none, since Merge-Review takes no amendment.
`contracts/stencils`' `rubricMarkerAllowlist` gains `amendment`, and `TestLoomRubrics`' doc and its per-rubric marker checks name it as the marker the four amendment-capable rubrics carry and the merge rubric does not.
`FullEnv` fills the new seams with fakes and `AmendMaxBounces` in its segment settings, so every recipe build in the tests keeps building.
`envkit`'s `nilLegal` gains the six new `Env` paths `DocumentSubject`, `CodeSubject`, `MergeSubject`, `MergeInOnly`, `WeftHead` and `RejectReview`, whose nil is a documented default: the first five's for a Bouncer row that names no subject, and `RejectReview`'s for a reject with no review file, so batten's wiring, which never fills a loom-only seam, and loom's wiring before card 15 keep passing the `NilSeams` audits of `TestWire_EverySeamFilled`.
`doc.go`'s "Segment settings and row settings" paragraph names the amendment allowance among the values a segment carries, and its review-segment-rows paragraph names the Bouncer's optional `subject` key and the Env seams each value selects.
`contracts/specs/shed-recipe-spec.md`'s Bouncer-row paragraphs, which describe `commit_seam`, `skip_seam` and `carry_over` as keys that select among the seams `Env` carries, gain `subject`, with its three values `document`, `code` and `merge`, the seams each selects, the construction error for a named seam the `Env` does not carry, and the construction error for an unknown value.

`TestBouncerEntry_Subject` is one table: each of the three values reaches the Bouncer with its seams, asserted from this package through two Calls: `layoutBouncerRound1Report` lays out round 1, a first Call over `judgeSeamShuttle` settles it CONVERGED and calls the row's named seam (counted by its fake, the document seam told the resolved artifact paths), and a second Call over a shuttle that fails the row when run returns Done with no spawn; `merge` builds a full generation; a missing named seam is refused naming it; an unknown value is refused; an absent key builds as before; `AmendMaxBounces` reaches the allowance.
`TestBurlerReviewEntry_RubricStencil` adds a row whose run directory holds an amendment generation record: the profile's rubric holds the note.

**ImpactSummary:** New optional Bouncer key and Env seams; no embedded recipe names the key yet, so every existing build is unchanged.

**Commit:** 13: shedrecipe: each Bouncer row names its subject seam and takes the amendment allowance
