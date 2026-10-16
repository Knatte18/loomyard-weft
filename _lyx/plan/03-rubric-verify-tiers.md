# Card 3 — rubric-verify-tiers

**Edit:**
- `internal/shedadapters#RubricNotes`
- `internal/shedadapters#ReadRubric`
- `internal/shedadapters#BouncerConfig`
- `internal/shedadapters#Bouncer.rubricNotes`
- `internal/shedadapters#TestReadRubric`
- `internal/shedadapters#TestBouncer_ClusterExcludesReachesSeedAndJudgePrompts`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#bouncerEntry`
- `internal/shedrecipe#burlerReviewEntry`
- `internal/shedrecipe#TestBouncerEntry_HappyPath`
- `internal/shedrecipe#TestBurlerReviewEntry_RubricStencil`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#TestWire_LanguageSkills`
- `contracts/stencils#rubricMarkerAllowlist`
- `contracts/stencils#TestLoomRubrics`
- `contracts/stencils/loom/loom-rubric-plan-review.md`

**Uses:**
- `plan:internal/languageconfig#VerifyTiersValue`

**Intent:** The Plan-Review rubric checks the plan's `## verify:` section against the hub's `verify_tiers` instead of this repository's Go tiers (D6).

- `RubricNotes` gains `VerifyTiers []string`, and `ReadRubric` fills a `verify_tiers` marker in its fixed value map with `languageconfig.VerifyTiersValue(notes.VerifyTiers)`, so the strict `stencil.Fill` never meets an empty value; a rubric without the marker ignores it, as for the other notes.
  `ReadRubric`'s doc and the `RubricNotes` field doc say only the Plan-Review rubric carries the marker and that the value is the entries as bullets or `none`, never parsed.
- `BouncerConfig` gains `VerifyTiers []string`, which `Bouncer.rubricNotes` copies into the notes, so the seed and judge passes both read it.
- `shedrecipe.Env` gains `VerifyTiers []string`; `bouncerEntry` sets `BouncerConfig.VerifyTiers` from it and `burlerReviewEntry` sets `notes.VerifyTiers` from it, so the Bouncer seed and judge rows and the Burler review row are told the list.
  `loomCLI.wire` sets `Env.VerifyTiers` from the `Language.VerifyTiers` Card 1 keeps; `battencli`'s `Env` runs no rubric row and is left unset.
- `loom-rubric-plan-review.md`'s `Verify coverage` bullet: a targeted package whose tests the section leaves unrun, its hermetic build-tagged tests included, is a finding;
  the hub's verify tiers follow as `{{.verify_tiers}}` on its own line; a tier the list says is compiled rather than run is not a finding, and a section out of the list's order is a finding;
  with `none`, only the coverage rule applies.
  The `go vet -tags llm`, `go vet -tags tmux` and `go -C` examples and the "order the plan template states" sentence leave the bullet.
  The header comment's marker sentence names `verify_tiers` among the markers `ReadRubric` fills.
- `contracts/stencils/rubric_test.go`'s file header comment, which lists the markers a rubric may carry and which rubric renders each note, names `verify_tiers` and that only the Plan-Review rubric renders it.

Tests:

- `TestReadRubric` gains rows over a body `Tiers:\n{{.verify_tiers}}\n`: two tiers render as two bullets in order, verbatim, one holding `{{.x}}`; no tiers render `none`; the row struct gains a `verifyTiers` column passed into `RubricNotes`.
- `TestBouncerEntry_HappyPath` sets `env.VerifyTiers` beside its webster record, its rubric line carries `{{.verify_tiers}}` beside `{{.webster_record}}`, and its existing seed-prompt check gains one `contains` for the tier bullet.
- `TestBouncer_ClusterExcludesReachesSeedAndJudgePrompts` sets `cfg.VerifyTiers` beside `cfg.WebsterRecord`, its rubric override carries `{{.verify_tiers}}`, and both its seed and judge checks assert the tier bullet, so the judge pass is pinned too.
- `TestBurlerReviewEntry_RubricStencil` gains a subtest `FillsTheVerifyTiers`, shaped like `FillsTheWebsterRecordNote`, asserting `profile.Rubric` carries the env's tiers as bullets.
- `rubricMarkerAllowlist` admits `verify_tiers`, and `TestLoomRubrics` passes a known tier list in `RubricNotes` and asserts only `loom-rubric-plan-review` renders its bullet.
  The ban on Go tier commands in the rubric's text is Card 4's one scan, not an assertion here.
- `TestWire_LanguageSkills`'s listed row asserts `c.env.VerifyTiers` equals the file's tiers.

**ImpactSummary:** Additive struct fields across shedadapters and shedrecipe plus one more fixed marker in `ReadRubric`; every other rubric renders unchanged because none carries the marker.
