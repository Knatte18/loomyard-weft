```yaml
lens: fix-consistency
outcome: reported
```

Checking the fix's agreement with its sibling cards, decisions and cross-references (fix-consistency lens).

**MEDIUM — card 23, bring-up paragraph (`isBeam` sentence, line 94).** The fix adds an `isBeam` seam that returns an error, but never says what `pauseOwnedRuns` does when that read fails or the seam is nil.
- For `endOwned`, the paragraph does say it: "a nil seam is skipped and warned as `pauseOwned` is today".
- For `isBeam`, nothing says whether a failed read warns and skips the end, ends without the dobby, or fails the bring-up.
- `TestStart_ReconcilesTheHubFirst` and `startHarness.runWith` assert the flag but have no case for a failed beam read.
- So the step that sets `endDobby` cannot be carried out without a further choice.

**LOW — card 23, `**Uses:**` list.** The rewritten text has `wireCLI` call `boardengine.ReadOwners`, but card 23's `**Uses:**` does not list `internal/boardengine#ReadOwners`.
- The card's other new external call, `internal/pairteardown#EndSubjectProcesses`, is listed.
- Card 15 lists `internal/boardengine#ReadOwners` under its own `**Uses:**` for the same kind of read.
- Evidence: grep of card 23 finds `boardengine` only at line 94.

**LOW — card 23, `**ImpactSummary:**`.** It still lists only removals ("one seam field leaves InnerRunDeps, dobbyengine.Deps and hibernateengine.Deps each"). It does not say that `orchCLI` swaps `pauseOwned` for `endOwned` and gains `isBeam`, which the rewritten bring-up paragraph now states.

Checked and consistent:
- Card 20 line 63 and card 21 line 55 describe the `DobbyPairSlugs` predicate the same way.
- Card 21's `handedOver` matches its daemon field list (line 44) and the "dropping the per-run memory" sentence.
- Card 21's statement that a pair left unregistered survives the hand-over matches card 17 step 2 and the Bound at line 27.
- Card 15's refusal-before-`Clear` matches card 22 lines 34 and 39 (intent pushed, then `lym tender resume` once green).
- Card 23's changed text matches the parent decision record entry of 2026-11-17, apart from where the seams are filled (`wireCLI`, not the `installedOwnerPause` path), which the fixer already raised with the parent.

Tool calls:
1. Grepped the plan cards and the decision record for twins of the changed terms. My context lacked the other cards and the record's added entries.
2. Grepped card 17's pair-removal text and card 23's `boardengine` references. My context lacked card 17, and I needed card 23's `**Uses:**` against the new call.
