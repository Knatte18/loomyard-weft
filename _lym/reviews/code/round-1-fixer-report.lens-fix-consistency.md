```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the F1–F4 change set against its surroundings.

- **LOW**, `_lym/landing/summary.md`, `## Tests` section (the file F4 edited): it lists only the two earlier tests and none of the rows F1 and F2 added.
  - The missing rows are `RoundWithNoTargetIdCountsAsChanged` in `TestBouncer_Stalemate`, `ARecordWithNoTargetIdsIsVerified` in `TestBouncer_FixVerification`, and `a target that matches no file leaves no target id` in `TestBurlerReviewProducer_Call_RecordsRoundSnapshot`.
  - The F1 rows are what pin the summary's own claim under `## Changes` that "a missing target can never produce a stalemate". The Tests section gives no sign that this claim is covered.

Everything else agrees:
- A search of `internal/shedadapters`, `internal/burlerengine`, `internal/shedrecipe` and the burler and shed contracts found no remaining twin that calls overlay target paths or changes-file paths worktree-relative. The other hits concern different paths.
- The header comments of the three test tables still hold with the new rows.
- The `Paths` comment now matches the `WorktreeRoot` field comment.

Tool calls:
1. One Bash call: grepped for "worktree-relative" wording and showed `git diff 060437e3c HEAD` for the test files. My context did not show the contracts and `internal/shedrecipe` hits, or the formatted test files as committed.
