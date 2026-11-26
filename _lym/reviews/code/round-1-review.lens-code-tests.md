```yaml
lens: code-tests
outcome: reported
```

The lens was code-tests: for each behaviour the diff adds or changes, find the test row that would fail if it broke.

**Behaviours and the tests that pin them**

1. `newFindings` lets a hardener round's new finding pass at any severity and in any file: `TestCheckReviewRound/NewFindingsBelowBlockingOnUntouchedFilesInAHardenerRoundPass` (internal/burlerengine/roundcheck_test.go, diff row at 005 line 228).
2. The `origin: parent` refusal still runs before the hardener exemption: `TestCheckReviewRound/NewParentFindingWithNoAskInAHardenerRoundIsRefused`.
3. `carriedFinding` lets a hardener round move a carried finding to any file, with or without a changes file: `CarriedMovedToAnUntouchedFileInAHardenerRoundPasses` and `CarriedMovedWithNoChangesFileInAHardenerRoundPasses`.
4. A carried finding's severity rule still holds in a hardener round: `CarriedSeverityRaisedInAHardenerRoundIsRefused`. The class rule has no hardener row. It sits above the exemption and the diff does not change it, so this is no finding.
5. Rounds that are not hardener rounds keep the BLOCKING and changed-file rules: existing rows such as `NewBlockingOnAnUntouchedFile` (roundcheck_test.go:152) and `CarriedPathMovedToAnUntouchedFile` (roundcheck_test.go:149).
6. The `validate-review` verb sets `HardenerRound` only in the hardener run directory: `FocusFindingAdmittedInTheHardenerRunDirectory` and `FocusFindingRefusedInTheCodeRunDirectory` (internal/burlercli/validate_integration_test.go:96-97). Both still kill a missing flag, because z.go is outside the changes file.
7. A `harden_round` BurlerReview row sets the flag and it reaches the round context through `roundProfile` and `BuildRoundContext`: `TestBurlerReviewEntry_ProfileMapping/HardenRoundMarksTheRoundContext` (entries_burlerhalves_test.go:329), `TestBuildRoundContext/HardenerRoundComesFromTheSources`.
8. `decisionRuleMarker` with `firstRaisedGates` renders round 1's rule from round 2 on and keeps CIRCLING at or above the checkpoint: `TestDecisionRuleMarker_CirclingOnlyFromTheCheckpoint`, rows "round 2 with first-raised keys gating" and "above the checkpoint with first-raised keys gating".
9. `RenderJudgePrompt` passes `BouncerConfig.FirstRaisedGates` through: `TestBouncer_JudgePromptGatesAFirstRaisedKeyWithFirstRaisedGates`.
10. `bouncerEntry` sets `FirstRaisedGates = true` for a `harden_round` Bouncer row (internal/shedrecipe/entries_bouncer.go:372): **none**.
11. Removing the `FocusOrigin` constant changes no behaviour.

**Findings**

- **MEDIUM · design** · internal/shedrecipe/entries_bouncer.go:372 · No test checks that a `harden_round` Bouncer row sets `FirstRaisedGates`, so deleting that line, which is the hardener's whole wiring of item 3, fails no test.
  - Evidence: the only test that names `FirstRaisedGates` sets it by hand on a `BouncerConfig` (bouncerprompt_test.go:110-123).
  - The `harden_round` Bouncer subtests in `TestHardenRoundKey` (internal/shedrecipe/entries_harden_test.go:187-240) check the refusal, the seed and the facts, never the judge prompt's decision rule.
  - Fix: add a row to `TestHardenRoundKey` that builds a `harden_round` Bouncer through `bouncerEntry`, runs round 2's judge call, and asserts the round-1 decision rule without the clause "on a key first raised in the latest round does not block it".
- **MEDIUM · design** · internal/burlerengine/roundcheck_test.go:224-227 (diff 005) · The diff removes `NewFocusDecisionBelowBlockingIsRefused`, `NewFocusFindingOfAnotherClassOnAnUntouchedFileIsRefused` and `NewFocusDecisionOnAnUntouchedFileOutsideAHardenerRoundIsRefused`, and no commit in the range carries a `Loosened-Check:` trailer (git log over 85ac0903e..HEAD).
  - The first one removes the assertion that a hardener round's new finding below BLOCKING is refused. Amended item 1 asks to exempt only the changed-file and changed-hunk rules, not the BLOCKING rule.
  - The third one's refusal is still covered by `NewBlockingOnAnUntouchedFile` (roundcheck_test.go:152), so it alone is not a loss.
  - Fix: either keep a hardener-round row that refuses a new finding below BLOCKING, or record a `Loosened-Check:` trailer naming the removed rows and the measured reason.
- **NIT · consistency** · internal/burlercli/validate_integration_test.go:96-97 · The row names `FocusFinding…` still describe the `origin: focus` exemption that the diff removes. The rows now test the hardener-round flag, not a focus-finding permission.
  - Fix: rename them to name the hardener run directory's exemption.

Tool calls:
- grep of shedrecipe tests for `harden_round` and judge-prompt checks, plus roundcheck_test "does not list" rows: the manifest held no shedrecipe Bouncer-entry tests or the base refusal rows.
- `sed` of entries_harden_test.go:150-240: to confirm that the `harden_round` Bouncer subtests never check the decision rule.
