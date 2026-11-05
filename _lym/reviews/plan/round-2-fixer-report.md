# Round 2 fixer-report — plan

F5 is `decision`-class and is left out here; the parent's recorded decision covers it.
`lym loom validate-plan` passes after every change, with only the informational findings the review notes at the baseline.

## Changes

- **F1** (fixed as proposed, plus one member the self-check found):
  - Card 24's Edit list gains `TestLoadConfig`, `baselineFixture.seedScopeRecord`, `TestDriftBaseline`, `TestDriftAwait`, `TestDriftVerify`, `newVerifyFixture` and `TestWire_DarnRecipe`, and its Uses drops `TestWire_DarnRecipe`.
    Its Intent says the `TestLoadConfig` subtest drops `BaselineReport` and `VerifyReport` from its durable map.
    `seedScopeRecord` drops its `report` parameter and its write of `baseline.md`, and its callers drop the argument.
    `TestWire_DarnRecipe`'s sub-run reads `ExpectTable("")` and keeps the `subject_reading` assertion.
  - Card 25's Edit list gains `TestLoadConfig`, and its Intent says the subtest drops `SideDir` and `Runs` from its ephemeral map.
  - Card 26's Edit list gains `baselineFixture` and `newBaselineFixture`, and its Intent says the fixture stops setting `Config.Skip`, so `configSkip` and the `TestDriftBaseline` row that sets it go.
  - `TestWire_DarnRecipe` leaves the Uses lists of cards 7, 8, 9, 12, 15 and 16.
- **F2** (fixed as proposed, with the test row corrected by the self-check): card 9's `CaptureRequest` gains `Sleep func(context.Context, time.Duration) error`, which is nil to sleep for real; `loomcli`'s `driftDeps` leaves it nil.
  `CaptureCurrent` waits through it from one second, doubling to thirty.
  The `TestCaptureCurrent` row's fake `Sleep` releases the lock on its second call, and the row asserts waits of one second, then two.
- **F3** (fixed as proposed, plus constants for the other rows): card 8 creates `expectRow`.
  Cards 8 and 10 call `PastRow(d.Rows, status row, expectRow)`, and card 10's Uses names `expectRow`.
  Card 19 creates `recheckRow`, `darnWriteRow` and `websterRow`, uses `expectRow`, and says `Amend` and `Defer` name their admitted rows through driftengine's own row constants.
- **F4** (fixed as proposed, plus two gaps the self-check found):
  - Card 15's `JudgeSnapshot.Changes` judges a weft path by content.
    A path that was uncommitted at the snapshot and whose content still matches the recorded hash or copy is no change, and neither is an uncommitted deletion that is still absent.
  - `driftDeps` adds the friction archive prefix to `LoomWeftDirs`, which now holds directories or prefixes ending in `-`.
  - `TestJudgeSnapshot` gains rows for:
    - a reviews file committed later with the same content;
    - changed content or a new path there;
    - a committed deletion;
    - an archive sibling path.
- **F6** (fixed as proposed, plus the rest of that sentence): card 22 names the overview's darn recipe sentence.
  That sentence now puts Drift-Expect between Drift-Await and Darn-Write and names the `drift` gate entry on Darn-Write.
  It also has Drift-Baseline record the scope from config with no agent, and has Drift-Verify and Drift-Recheck check containment by code and the judge.

## Self-check

- fix-claims, MEDIUM: card 24's list of `seedScopeRecord` callers omitted `newVerifyFixture`. **Fixed as proposed:** added to Edit and to the caller sentence, which now reads "wherever card 16's rewrite kept the call".
- fix-consistency, MEDIUM: `expectRow` duplicates `loomshed.NameDriftExpect`, and card 19's constants had no owning card. **Fixed with my own change:** I did not switch to `loomshed`'s names, because driftengine keeps its own row constants (`verifyRow` in `verbs.go`, `captureRowBaseline` in `checkmain.go`) beside `loomshed`'s and imports no `loomshed`; fix-proposal confirmed this. Instead card 19 creates `recheckRow`, `darnWriteRow` and `websterRow`, so a card owns every constant it names.
- fix-consistency, LOW: the overview sentence card 22 names stays stale on Drift-Baseline and on Drift-Verify/Drift-Recheck "compare". **Fixed as proposed.**
- fix-consistency, LOW: card 15's `LoomWeftDirs` definition did not cover the archive prefix. **Fixed as proposed:** it is now defined as directories or prefixes ending in `-`, and the exemption sentence reads from it.
- fix-consistency, LOW: cards 10 and 19 used `expectRow` without listing it under Uses. **Fixed as proposed.**
- fix-proposal, MEDIUM: card 9's test row could not observe two waits if the first `Sleep` released the lock. **Fixed as proposed:** the lock is released on the second call.
- fix-proposal, LOW: the content rule did not cover an uncommitted deletion. **Fixed as proposed:** an uncommitted deletion that is still absent is no change, and a `TestJudgeSnapshot` row covers it.
- fix-proposal, LOW: card 24's `seedScopeRecord` callers. A duplicate of the fix-claims finding, fixed there.
- fix-affected-set, LOW: `TestDriftEntries`' `VerifyNilCompareAccepted` row was left on no card. **Fixed as proposed:** card 16 drops it. The Recheck rows go with card 23.

## Disputed

None.

## Deferred

None.
