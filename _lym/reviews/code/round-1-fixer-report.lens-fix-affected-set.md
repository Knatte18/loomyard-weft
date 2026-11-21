```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set lens. I reran the set searches for F8, F3, F4 and F2 and left F1 and F11 unchecked.

**Findings**

1. **LOW, F3.** `internal/fabricengine/mergestate.go:35` still says "A record without them, written by an older binary, reads them as empty."
   - The fix deleted both consumers of that case: the `sourceSHA == ""` exemptions in `mergelifecycle.go` and `mergeguards.go`, and the matching `doc.go` text.
   - A record with an empty source SHA now fails in `repo.IsAncestor` instead of being exempt. That is the "invalid SHA" error `TestMergeCrucible_ContinueRefusesAttemptThatNeverReachedBothSides` hit before the fixer supplied `WarpSource`.
   - The comment describes compatibility behaviour that no longer exists.

2. **LOW, F3.** The review's "drop ScopePartial… in config.go, baseline.go and verdict.go" is not met in `verdict.go`. The fixer kept the stored-record reader as a deliberate choice, but it was not recorded as a disposition.
   - `internal/driftengine/verdict.go:102` still has `case ScopePartial` ("partial, an old scope…").
   - `internal/driftengine/check.go:279` still has `if scope.Scope == ScopePartial`.
   - `internal/driftengine/records.go:20-21` still declares and documents the constant.
   - The fixer-report should list this under `fixed-differently`.

3. **No finding for F8, F4 or F2.**
   - F8: the search for retired-rename, legacy `self`, one-time strip and `migrateRunSettings` wording found no remaining match in docs, pattern, contracts or non-test Go, apart from an unrelated `shedrun` comment.
   - F4: the only remaining `.fabric-anchor` mentions are `lymcwd/anchor.go` and the `clone.go:453` comment. No doc still prescribes `git mv .fabric-anchor`.
   - F2: the other `loom: {plan_max_bounces…}` occurrences are test fixtures in `yamlengine` and `runprofile`. They don't go through a loom `ConfigSpec` check. The `configreg/runsettings_test.go:411` row still passes because its refusal text contains `plan_max_bounces`.

**Tool calls**
1. One Bash call with the four greps (F8, F3, F4, F2). Context lacked repo-wide search results for these sets.
