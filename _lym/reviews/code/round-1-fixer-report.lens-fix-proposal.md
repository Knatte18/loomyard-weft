```yaml
lens: fix-proposal
outcome: reported
```

No findings.

Challenges built against the F1 proposal, none of which held:
- **Lock-directory row:** `ReadOriginFor` stats the record first, and `MarkBeam` has written it, so the row reaches the missing-lock-directory error at `origin.go:134`. `isBeam` then returns false and `Spawn` opens the bare folder.
- **Absent-record row:** `ReadOriginFor` returns `found == false` with no error. This row follows the lock-directory row, so the lock directory has to be restored first, and the test restores it.
- **State left behind:** `.weft/` is in the weft worktree's exclude list (`recordsgit.go:111`), so removing and recreating it leaves no git-visible change. The lock file is created on demand, so later steps are unaffected. The removed origin record leaves the pair unmarked. The prime and hub regeneration steps that follow treat an absent record as a task, and tier1 passed with the new step in place.
- **Defect in place:** if `isBeam` returned true on error or on an absent record, `Spawn` would regenerate and launch the beam workspace path. That fails both the launch-path assertion and the "no workspace file written" assertion.

Calls made:
1. `grep` for `weftLockDirName` and `originRecordLockFileName` in `internal/fabricengine/*.go`: to see whether `.weft` is git-excluded and the lock file is created on demand. My context did not hold `recordsgit.go`.
