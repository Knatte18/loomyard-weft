# Card 7 — bouncer-review-inputs

**Create:**
- `plan:internal/shedadapters#ReviewInput` -> `type ReviewInput struct`
- `plan:internal/shedadapters#ReviewInputNote` -> `func ReviewInputNote(runDir string) string`
- `plan:internal/loomcli#reviewInputs` -> `func reviewInputs(scratchDir string) func() ([]shedadapters.ReviewInput, error)`
- `plan:internal/loomcli#prFindingsFile` -> `func prFindingsFile(scratchDir string, n int) string`

**Edit:**
- `internal/shedadapters#BouncerConfig`
- `internal/shedadapters#Bouncer.keptReviewInput` -> `func (b *Bouncer) keptReviewInput(settledAt string) ([]ReviewInput, error)`
- `internal/shedadapters#Bouncer.startNextGeneration`
- `internal/shedadapters#Bouncer.reenterUnchanged`
- `internal/shedadapters#generationRecord`
- `internal/shedadapters#AmendmentNote`
- `internal/shedadapters#RubricNotes`
- `internal/shedadapters#ReadRubric`
- `internal/shedadapters#Bouncer.rubricNotes`
- `internal/shedadapters#TestBouncer_Amendment`
- `internal/shedadapters#withReviewInput`
- `internal/shedadapters#amendmentInputRow`
- `internal/shedadapters#reentryInputRow`
- `internal/shedadapters#TestReadRubric`
- `//internal/shedadapters/doc.go`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_ReviewKeysReachTheEnv`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#bouncerEntry`
- `internal/shedrecipe#burlerReviewEntry`
- `internal/shedrecipe#TestBurlerReviewEntry_RubricStencil`
- `internal/shedadapters#BurlerHalfDeps`
- `internal/shedadapters#BurlerReviewProducer`
- `internal/shedadapters#NewBurlerReviewProducer`
- `internal/shedadapters#BurlerReviewProducer.Call`
- `internal/shedadapters#BurlerReviewProducer.roundProfile`
- `internal/shedadapters#TestBurlerReviewProducer_Call_ProfileAndOpts`
- `internal/shedadapters#halfProducerSpec`
- `internal/shedadapters#newReviewProducer`
- `internal/shedrecipe#burlerRoundProfile`
- `//contracts/specs/shed-recipe-spec.md`
- `internal/testkit/envkit#FullEnv`
- `internal/testkit/envkit#nilLegal`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#fixerReportInput`
- `internal/loomcli#TestReviewInputSeams`
- `contracts/stencils/loom/loom-rubric-code-review.md`
- `contracts/stencils#rubricMarkerAllowlist`
- `contracts/stencils#TestLoomRubrics`

**Uses:**
- `internal/loomcli#reviewInputFile`
- `internal/loomengine#LoomScratchDir`
- `//internal/shedrecipe/doc.go`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestFullEnv_PathFieldsAbsoluteUnderOneRoot`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/testkit/envkit#TestSeamMarkers_ReflectiveTestsAreMarked`

**Intent:** D4's re-entered Code-Bouncer takes a send-back file as a review input that forces a full generation, and D6's rubric section reads it.

The seam:

- `ReviewInput` is `Path`, `WrittenAt` and `Full`; `Full` marks an input that forces a full generation.
- `BouncerConfig.ReviewInput` and `shedrecipe.Env.ReviewInput` become `ReviewInputs func() ([]ReviewInput, error)`, every input there is; nil stays the absent value.
  `bouncerEntry` passes it on for a row with a subject, as today; `envkit`'s `FullEnv` fills it with an empty answer, and its `nilLegal` key `"ReviewInput"` becomes `"ReviewInputs"`, so `NilSeams` keeps skipping the field.
  The test helpers that build or read the seam follow the new shape: `withReviewInput` builds a one-input list, `amendmentInputRow` and `reentryInputRow` read the record's `ReviewInputs`, and `TestWire_DarnRecipe` and `TestWire_ReviewKeysReachTheEnv` check and call `ReviewInputs`.
  `doc.go`'s passages on the review input describe the list, its `Full` flag and the full generation it forces.
  `shed-recipe-spec.md`'s Bouncer passage names `Env.ReviewInputs`, which stays optional, says the Bouncer takes every input written after the settle, and says an input marked full forces a full generation.
- `loomcli`'s `reviewInputs(scratchDir)` answers the Parent-Align fixer report, through `fixerReportInput`, as a non-full input, and the newest `pr-findings-<n>.md` under `scratchDir`, the highest `n`, as a full one; `loomCLI.wire` sets it in place of `fixerReportInput` alone.
  `fixerReportInput` keeps its fallback to the retired name.
  `prFindingsFile(scratchDir, n)` is the one spelling of `<scratchDir>/pr-findings-<n>.md`, which card 8's verb writes.

The Bouncer:

- `keptReviewInput(settledAt)` returns every input written after `settledAt`, in the seam's order.
- `reenterUnchanged` on an unchanged subject re-opens it when any input was kept: a full one whatever `FullGeneration` says, a non-full one only without `FullGeneration`, as today.
  With a full input kept it starts the full generation itself, through `startNextGeneration`, and reports handled true, so a settle record at round 0, as an amendment approved with an allowance of 0 leaves, never falls through to the `n == 0` seed path with no generation record naming the input.
- `startNextGeneration` starts a full generation, never an amendment, when a kept input is full, even on an unchanged subject, so the round reviews the whole range with the perch's own budget and hard cap.
  Otherwise it decides as today, and an amendment keeps the non-full inputs.
- `generationRecord` gains `ReviewInputs []string` (`review_inputs`, omitempty) holding each kept path, full or not; a record written before, holding only `review_input`, reads as that one path.
- `AmendmentNote` keeps naming an amendment's review input, the Parent-Align fixer report, as today.
- `ReviewInputNote(runDir)` renders, for a full generation whose record holds inputs, `This generation reviews the whole range; it was opened by review inputs written after the perch last settled, which you read besides the diff:` and one line per path; it renders `none` otherwise.
- `RubricNotes` gains `ReviewInput`; `ReadRubric` fills the `review_input` marker from it, `none` when blank; `Bouncer.rubricNotes` sets it from `ReviewInputNote` at each call, beside the amendment note.

The review row renders its rubric at each round.
`lym loom run` builds the shed once, so a rubric filled when `burlerReviewEntry` builds the row holds the note of the generation open when the process started;
a send-back rerouted inside that process would reach the reviewers as `none`.

- `BurlerHalfDeps` gains `Rubric func() (string, error)`, which `NewBurlerReviewProducer` stores; nil keeps the profile's rubric, as a literal-rubric row does.
- `BurlerReviewProducer.Call` reads it before it builds the round's profile, and `roundProfile` sets `Profile.Rubric` to its answer.
  A failed read is an error of the call, returned before the profile is built, so no reviewer is spawned.
- `burlerReviewEntry` sets it on a `rubric_stencil` row to a closure calling `ReadRubric` with the notes it builds today, `Amendment` and `ReviewInput` read through `AmendmentNote(runDir)` and `ReviewInputNote(runDir)` at each call.
  The fill at build stays, so a bad stencil still fails construction.
- The doc comments of `NewBurlerReviewProducer`, `roundProfile` and `burlerRoundProfile` name the rubric among the per-round fields, set from the seam's answer.

The rubric, `loom-rubric-code-review.md`'s `## Review input` (D6): it carries `{{.review_input}}`, and reads each input that note or the amendment note names.
The fixer report keeps today's rule.
A PR-findings input, a `pr-findings-<n>.md` file of findings PR-Review received after Code-Review approved: review the whole range as usual, read the file, and file each of its findings that holds against the code as a finding of this round, never a finding about the file.
`rubricMarkerAllowlist` admits `review_input`, and `TestLoomRubrics` fills it and asserts the Code-Review rubric carries the note.

Tests, tier0:

- `TestBouncer_Amendment` gains rows: a send-back input written after the settle, on an unchanged subject, starts a full generation whose record holds the input's path and whose `ReviewInputNote` names it (criterion D4);
  the same after an amendment with an allowance of 0 settled at round 0;
  the same with `FullGeneration` true;
  a send-back input and a fixer report both kept start a full generation holding both;
  a fixer report alone still starts an amendment, as today;
  a generation record written with only `review_input` reads as that one path.
- `TestReviewInputSeams` asserts `reviewInputs` answers nothing with no file, the fixer report as non-full, and of `pr-findings-1.md` and `pr-findings-2.md` only the second, as full.
- `TestReadRubric` gains the `review_input` note.
- `TestBurlerReviewProducer_Call_ProfileAndOpts` gains rows, through a new `rubric` field of `halfProducerSpec` that `newReviewProducer` sets as `BurlerHalfDeps.Rubric`: a `Rubric` seam whose answer differs from the template's rubric sets the round's `Profile.Rubric` to that answer;
  a failing `Rubric` seam fails the call and spawns no reviewer.
- `TestBurlerReviewEntry_RubricStencil` gains a row (criterion D4) that builds the Code-Review row over `loom-rubric-code-review.md` with `burlerReviewEntry` itself, in a run directory with no generation record, and leaves `captureReview` untouched.
  After the build it writes a full generation's record naming a `pr-findings-1.md` path among its review inputs, then calls the producer.
  It reads the rubric from the `BurlerHalfRunner` fake's `GotReviewProfiles` and asserts it names that path.

**ImpactSummary:** The review-input seam becomes a list with a full-generation flag across shedadapters, shedrecipe's Env, envkit and loom wiring; one new rubric marker; the burler review row renders its rubric at each round; Parent-Align's amendment path is unchanged.

**Commit:** `7: bouncer-review-inputs`
