```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set searches behind each finding whose premise is a set (F1, F3, F4, F5), and check that the fix covers every member.

Unchecked findings: F2. Its premise is one row and one step, not a set, so there was no search to rerun.

no findings

- **F1 (lock acquisitions and `ArchiveOwnerTips` use):** card 3 no longer names `ArchiveOwnerTips` anywhere except in the sentence that rules it out. Card 9 is the only other card that uses `TakeOverRun`, through `TakeOver(slug, subject)`. It sets no `LockWait`, so the zero default applies, which is consistent.
- **F3 (deps constructions that must set the gained fields):** `checkDeps` is built at `check.go:465`, which the fixed text addresses, and at `check_test.go:63`, which builds its fields by hand. `fixDeps` is built at `fixloop.go:385`, which is addressed, and at `fixloop_test.go:52`, which belongs to `fixFixture` and is in the card's Edit list. No construction is missed.
- **F4 (consumers of the Hold seam's outcome):** card 9 reads `HoldLost` and the result's line, which `09-adopt-foreground.md:211` already says "names that machine or says the hold cleared". That agrees with the new step 3 wording. No other card reads the seam.
- **F5 (`Dir` uses):** `greenDeps` is built only at `green.go:73` (`hubGreenDeps`) and `green_test.go:41` (`greenFixture`). The fixed text covers both, plus the fixture's `dir` and `otherDir` fields.

Tool calls:
1. Grepped the code for `greenDeps{`, `checkDeps{` and `fixDeps{` constructions, and the plan for `ArchiveOwnerTips`, `LockWait`, `TakeOverRequest` and the Hold seam wording. My context held the field uses, not every place these structs are built or the other cards' references.
2. Grepped the other plan cards for `no_hold`, `runHoldCheck`, `holdCheckDeps`, `HoldLost`, `TakeOverRun`, `LockWait` and `archiveOwner`. My context held only cards 3 and 8.
