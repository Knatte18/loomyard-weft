```yaml
lens: fix-affected-set
outcome: reported
```

No findings. I checked each finding's set and the fix covers every member:
- **F1:** `RunView.EntryEnded` and `RunView.PairPresent` are set in one place, `DobbyRuns` in `internal/battencli/dobbyreads.go`. The extended `TestDobbyRuns_ReadsTheLandOutcomeAndTheLandingCommit` asserts both fields for every slug that function builds. Two things I did not check: that the `"pairless"` slug (pair deleted from disk after `AddPair`) is still read, and that `"landed"` covers the done-entry-with-pair case. The tier1 run passed.
- **F2:** the nil-seam guard has one site, the `d.deps.TearDownEnded == nil` check in `tearDownEnded`. The `d.teardown == nil` guard around `CheckBinary` also has one site, in `Run`. The nil-seam subtest and the binary-check subtest each pin one of them.
- **F3:** the retry ceiling is applied at one `min(...)` in `readEndedTeardown`, and the forgetting loop is one `delete` in `tearDownEnded`. The extended failing-teardown subtest and the extended removed-pair subtest assert one each.

Tool calls: 1. A `grep` for `PairPresent`, `EntryEnded` and `TearDownEnded` in non-test code, to confirm no second site sets the `RunView` fields and no second consumer of the guards exists. My context did not show the rest of the repo.
