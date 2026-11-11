```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-consistency lens over the change set (siblings and twins of the changed text).

Findings:

1. **LOW**, `internal/gatecli/tiers.go:120` and `:138` (the other plan-time notices in `planTierRuns`). The F5 fix defers only the run-whole notices to `printRunWholeNotices`. The "scope selected no test class, so %s runs whole" notice and the watch-skip notice still print inside the loop. A later refusal (a guard step under dotnet, a vet step failure) leaves those stale notices before the error. This is the same defect F5 names, left in its twin lines. Neither the code nor the `planTierRuns` doc comment says why only one notice kind was deferred. This may be acceptable scope, but a reader would see the inconsistency.

2. **LOW**, `internal/gatecli/tiers.go:80-82` (doc comment). It still says "A tier a no-test-class scope runs whole is named on request.stderr" and the run-whole sentence as unconditional. Nothing in the doc says the run-whole notice now prints only once no refusal can follow. `doc.go` is similarly silent. The new `printRunWholeNotices` comment states it, but the exported-behaviour doc does not.

Checked and consistent:
- The F1 refusal string matches the `tiers_test.go:493` exact row.
- `doc.go:144` still holds ("names ... the prime's `lym config gate` step"), and `doc.go:232` (role clause) now holds for this refusal too.
- No other file pins the old "only the prime adds" text.
- The `doc.go:192` reach sentence agrees with `namesProject`.
- `lowestTierReaching` and `runReachesProject` both go through `namesProject`, so F3 and F4 reach both callers.

Further tool calls: 1 grep/sed (the old refusal strings, run-whole notice text and `doc.go` lines 138-146), to find sibling copies the context lacked.
