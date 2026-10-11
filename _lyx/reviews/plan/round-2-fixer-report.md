# Fixer report — Plan-Review round 2

All five findings are fixed.
`lyx loom validate-plan` passes over the fixed plan (`ok: true`).
Its findings are informational only: the `create-new-unit` notes for card-created files, and the `.Call` name match in `internal/proc/namedjob_windows.go` that round 1 already reported.

## Changes

- **card4-loomcli-test-budget-row (fixed as proposed).**
  The premise holds: `cmd/lyx/testdata/test-budget.yaml` holds `internal/loomcli: 200`, the package has 200 top-level tests outside `TestMain`, and the file counts tests "under every build tag".
  Card 4's Intent now says the `internal/loomcli` row rises by the new smoke test, beside the `internal/shedrecipe` sentence.

- **card10-prompt-test-helpers-undecided (fixed as proposed).**
  The premise holds: `testRoundFilePaths` is a `roundFilePaths` (`internal/burlerengine/prompt_test.go:33`), and `seatPrompts` and `combinedPrompt` take a `roundPrompts` (`:75`, `:90`), and card 10 deletes both types.
  The decided list:
  - `testRoundFilePaths` and `seatPrompts` move to `**Delete:**`; the tests take paths from card 2's `reviewFilesIn` or `fixFilesIn`.
  - `chairPrompt` is a new `**Create:**` helper, since a half's table has one chair seat where `seatPrompts` returned two seats.
  - `combinedPrompt` keeps its name and takes a re-sign arrow: `func combinedPrompt(t *testing.T, stencilsDir string, p *Profile, table seatengine.Table, prompt halfPrompt) string`.

  The Intent's "drop whichever of its helpers no remaining test uses" is replaced by that list.
  Every caller of the two deleted helpers is a test card 10 already edits.

- **card2-validatefix-refusals-untested (fixed as proposed).**
  The premise holds: no card names a test that reaches a `validateFix` refusal, and `TestProfile_Validate` (`internal/burlerengine/profile_test.go:81`) is the existing table-driven validation test.
  Card 2 now lists `internal/burlerengine#TestProfile_Validate` under `**Edit:**`.
  Its Intent gives the test the proposed `validateFix` rows: an empty fixer-report path, identical review and fixer-report paths, an illegal `FixScope` and a missing target path, each refused, and a profile with no fasit or rubric, accepted.
  Card 10's sentence on that test now keeps card 2's `validateFix` rows.

- **card3-round-complete-ignores-acceptance (fixed differently).**
  The premise holds: `roundComplete` (`internal/shedadapters/burler.go:138`) stats the review and the fixer report only, and D3 says the complete-round check reads the acceptance record.
  The proposed placement, card 3, would break the wired old round: `BurlerProducer` calls `highestCompleteRound` and `hydrationPaths` (`burler.go:274`, `:315`), and it stays the production producer until card 4 and exists until card 10.
  It writes no acceptance record, so with the check in card 3 none of its rounds would ever count as complete, and its own tests, which build rounds with `writeRoundPair`, would fail.
  So the change lands in card 10, once no producer that writes no record is left:
  - `internal/shedadapters#roundComplete` and `internal/shedadapters#writeRoundPair` join card 10's `**Edit:**` list, with `plan:internal/shedadapters#readAcceptance` under `**Uses:**`;
  - `roundComplete` also requires the record, and `writeRoundPair` writes it with the pair, so the row tests' complete rounds stay complete;
  - `internal/shedadapters/burlerreview_test.go`, which card 3 creates, joins card 10's `**Edit:**` list, and gains the proposed row where a review and fixer report without a record are archived and reviewed again.

  Card 10's ImpactSummary names the behavior change.
  The proposed alternative, a Shared Decision keeping the pair, was not taken, since it would depart from D3's text.

- **card4-env-darn-burler-doc (fixed as proposed).**
  The premise holds: `internal/shedrecipe/recipe.go:216` says loomrecipe stamps the attempts "onto the verify gates of the Darn-Write and Darn-Burler rows".
  Card 4's `Env` sentence now says `DarnVerifyAttempts`'s doc names `Darn-Burler-Fix` in place of `Darn-Burler`.
  `shedrecipe#Env` is already a card 4 target.

## Disputed

None.

## Deferred

None.
