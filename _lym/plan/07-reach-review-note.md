# Card 7 — reach-review-note

**Create:**
- `plan:internal/loomcli#reachReportNote` -> `func reachReportNote(location *lymcwd.Location) func() string`
- `plan:internal/loomcli#renderReachReportNote` -> `func renderReachReportNote(report gatecli.ReachReport, found bool, matchesHead func() (bool, error)) string`
- `plan:internal/loomcli#TestRenderReachReportNote` -> `func TestRenderReachReportNote(t *testing.T)`
- `plan:internal/shedadapters#reachReportMarker` -> `const reachReportMarker = "reach_report"`
- `plan:internal/shedadapters#reachReportNote` -> `func reachReportNote(rubric []byte, read func() string) (string, error)`

**Edit:**
- `internal/shedadapters#RubricNotes`
- `internal/shedadapters#ReadRubric`
- `internal/shedadapters#BouncerConfig`
- `internal/shedadapters#Bouncer.rubricNotes`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#bouncerEntry`
- `internal/shedrecipe#reviewRubricNotes`
- `internal/loomcli#loomCLI.wire`
- `internal/testkit/envkit#nilLegal`
- `contracts/stencils/loom/loom-rubric-code-review.md`
- `contracts/stencils#rubricMarkerAllowlist`
- `internal/shedrecipe#TestBurlerReviewEntry_RubricStencil`
- `internal/shedrecipe#TestBouncerEntry_HappyPath`
- `internal/loomcli#TestWire_ReviewKeysReachTheEnv`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestFullEnv_PathFieldsAbsoluteUnderOneRoot`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/testkit/envkit#TestSeamMarkers_ReflectiveTestsAreMarked`

**Extends:**
- `internal/loomcli/doc.go`
- `internal/shedadapters/doc.go`
- `internal/shedrecipe/doc.go`

**Uses:**
- `plan:internal/gatecli#ReadReachReport`
- `plan:internal/gatecli#ReachReportMatches`
- `plan:internal/gatecli#ReachReport.NoteLines`
- `internal/loomcli#redundancyReportNote`
- `internal/shedadapters#redundancyReportNote`

**Intent:**
Give the code review rubric a `reach_report` marker beside `redundancy_report`, filled as that one is (D5).

- `reachReportNote(location)` returns the closure the wiring hands the review: at each fill it reads `gatecli.ReadReachReport`,
  and `renderReachReportNote` renders it: the report's note lines (`ReachReport.NoteLines`) under one line saying which gate call wrote it, when `matchesHead` holds for HEAD;
  otherwise one line opening "No current reach report" that says why: none written, a red tier (naming the gate key), outside a hub or under another scoper (the report's note), unreadable, unmatchable, or written for another tree.
  `matchesHead` is `gatecli.ReachReportMatches` against `Fabric.HeadSHA`, called only for a report that judged.
- `shedadapters`: `RubricNotes` gains `ReachReport func() string`; `ReadRubric` fills `reach_report` through `reachReportNote`, which renders `none` for a nil closure and errors for a rubric lacking the marker only as `redundancyReportNote` does;
  `BouncerConfig` gains `ReachReport`, and `Bouncer.rubricNotes` passes it.
- `shedrecipe.Env` gains `ReachReport func() string`, nil rendering `none`; `bouncerEntry` and `reviewRubricNotes` pass it as they pass `RedundancyReport`.
  `envkit`'s `nilLegal` lists `ReachReport`, since nil is its documented default.
- `loomCLI.wire` sets `ReachReport: reachReportNote(location)` beside `RedundancyReport`.
- The rubric (`loom-rubric-code-review.md`) gains a `## Reach report` section after `## Redundancy report`, holding `{{.reach_report}}` and a paragraph:
  the note is the advisory report `lym gate test` writes on the diff's changed functions, read only when it matches HEAD's tree;
  read it as evidence for "a behavior the diff adds with no coverage" and for naming a covering test in a redundancy finding;
  an unreached function is a candidate, never a finding by itself; with `none` or no current report, raise no finding about its absence.
  The test-economy bullet that names the redundancy report gains a clause pointing at the reach report for those two findings.

Tests:
- `TestRenderReachReportNote` (tier0) is one table in the shape of `TestRenderRedundancyReportNote`: no report, a red-tier report naming its key, a judged-nothing report with its note, a match error, a report for another tree, and a matching report rendering its note lines.
- `rubricMarkerAllowlist` gains `reach_report`, so `TestLoomRubrics` fills it.
- `TestBurlerReviewEntry_RubricStencil` gains a subtest beside `FillsTheRedundancyReportNote` asserting the reach note reaches the filled rubric; `TestBouncerEntry_HappyPath` asserts the same for the Bouncer.
- `TestWire_ReviewKeysReachTheEnv` asserts the wired `ReachReport` closure is set and returns a note naming no current report for a worktree with none.
- The seam-marked wiring and envkit tests listed under Edit keep passing with the new `Env` field; each gains only what its reflective walk needs.

`loomcli/doc.go`, `shedadapters/doc.go` and `shedrecipe/doc.go` gain the marker beside their `redundancy_report` sentences.

**ImpactSummary:** One rubric marker threaded from loomcli through shedrecipe and shedadapters into the code review rubric, mirroring `redundancy_report`; nil renders `none`, so every existing caller that leaves it unset is unchanged, and the seam-marked tests over `Env` are listed.
