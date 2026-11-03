```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1–F3 change set for consistency with its surroundings.

no findings

- The changed `checklistPoll` signature has one production caller (`internal/loomcli/checklist.go:127`) and one test caller (`internal/loomcli/checklist_test.go:1287`). Both pass the new second argument.
- The `checklistPoll` doc comment now says "wait line", which matches `checklistWaitLine`. The file header's "wait line carrying what the run waits on in full" still holds.
- The doc comment says a nil `waiting` reports none, and the `waiting != nil` guard in the body agrees.
- `status_test.go`'s file header and the `statusWatchLine` doc already describe the key and the unavailable branch. The new assertion matches them.
- The F1 stub's `Steady` is non-empty while `Text` is empty in the empty-note subtest. Only `Text` is checked there, as in `statusCmd`, so the subtest stays consistent.

Tool calls:
1. `grep -rn "checklistPoll(" --include=*.go .` — to confirm no caller of the changed signature was missed. My context only showed the earlier grep, which used a different pattern.
