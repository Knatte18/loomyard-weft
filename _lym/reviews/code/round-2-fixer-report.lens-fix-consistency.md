```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1 fix (deleted per-helper request test, its exports and the `time` import) for consistency with its surroundings.

no findings

- The deleted test name and both removed exports have no remaining reference outside `.git`, `.lym` and `_lym`, so no doc, comment or test still points at them.
- The `driftwiring_test.go` file header describes the detached capture child's start request. The remaining test, `TestCaptureSpawnRequest_StartsInTheAnchoredDirectory`, covers exactly that, so the header still matches.
- `export_test.go` keeps `CaptureSpawnRequest`, which the remaining test uses.
- The imports `driftengine`, `lymcwd`, `filepath` and `tiergate` are all still used by that test, and `go vet` and the gate were green after the edit.
- The commit touches only the two files the finding names (27 deletions).

Tool calls:
1. grep for the deleted names across the repo, excluding `.git`, `.lym` and `_lym`: my context showed only `internal/`, not docs or other trees.
2. `git show --stat HEAD`: to confirm the commit's file set.
