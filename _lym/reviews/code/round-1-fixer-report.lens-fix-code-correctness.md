```yaml
lens: fix-code-correctness
outcome: reported
```

No findings. I traced each changed branch by hand and ran no further tool calls.

- **`endsAsFamilyProject` (profile, run settings, notify_on_land guard):**
  - `arrasProjectOf` goes through `withoutRunFields`, which clears all three fields, so a legitimate `convertRunEntry` result still passes.
  - A write that keeps any of them used to pass, because `scopeChanged` ignores them. It is now refused, since the entry's kind also changed.
  - The three new lock rows (`AKeptProfileIsRefused`, `KeptRunSettingsAreRefused`, `AKeptNotifyOnLandIsRefused`) would fail if the guard were removed.
- **`Call` clearing `entry.StatusProducer` after the repair (F2):**
  - `repaired` implies the filing was found as its own, either with the store done or with the store write lost.
  - In both cases `stageAndFile` runs store-only or not at all, so skipping the claim is right.
  - The new loomshed row would show a `claim_status` step, and fail, if the clear were removed.
  - No other reader of `entry` after that point uses `StatusProducer`.
- **New `boardengine` tests:**
  - The `repointDependents` test hits the dangling-sink branch of `validateWrite`; `NewStore("")` has no vocabulary, so nothing else fires first.
  - The no-former-record row exercises the `os.Remove` restore branch.
  - The no-op `ClearProjectRunStatus` checks cover an absent entry, a non-project and a project with no status, and the `backdateBoardFile` assertion would catch a write.
  - Removing the draft guard from `endsAsFamilyProject` would make `AReleasedPartIsRefused` fail.
  - `APartTheBoardAlreadyHeldIsRefused` and `NoAddedPartIsRefused` each fail if the family-set or the empty-parts check is dropped.
- **`TestCommitStatusDeps_SkipsTheBoardStatusAfterAFilingMark`:**
  - The mark is written with `slug`, and `raddleFilingMarked` resolves `SelfRunID` to `location.WorktreeName`, which is that slug, so both sides use the same key.
  - Always wrapping the write fails row 3, and never wrapping it fails row 2.
- **`AFailingMarkOrStatusClearStopsTheRepairBeforeAnythingElseRuns`:** the path is `repairFamily`, then `fail`, which returns an error with an empty outcome. The `mark_filed` row stops before `clear_status`, as asserted.
- **Disputed F7:** the `parseErr != nil` early return at `internal/loomshed/raddlefile.go:205` comes before `finish`, so the premise is false and no change is needed.

Calls made: none. My context already held every changed file and the gate output, and the gate passed.
