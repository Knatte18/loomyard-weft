```yaml
lens: fix-consistency
outcome: reported
```

No findings.

Checked, and found consistent:
- **`deps.go:347`:** the `TaskHasCommits` field doc ("reports whether the task branch holds a commit the local parent branch lacks, read by git") matches the new `HeadHoldsCommitsNotIn` closure and its updated comment.
- **Error prefix:** the new method's `fabricengine: HEAD commits not in %s: %w` follows its siblings' `fabricengine: <op> ...: %w` form. The F1 test's "error names the parent branch" assertion holds because the wrapper includes `ref`.
- **Other `CommitLog` callers:** none outside `landingdeps.go:334` and `peers.go:289` computes a boolean from its length. They all use the lines, so there is no twin of the F4 change left behind. `CommitLog` still serves `CommitsMissing`.
- **Seam registries:** `wiring_seams_test.go:44` lists `Landing.PullRequest.TaskHasCommits` as filled in `run.go`, and `battencli/wire_seams_test.go:101` as loom-only. The closure change leaves both unchanged and accurate. The `envkit` default `(true, nil)` is also unaffected.
- **New test fields:** `taskErr` and `wantErrText` were added to both tables, and the call sites and error assertions match in both. Existing rows leave `wantErrText` empty, so the `Contains(..., "")` check passes for them.
- **F3 twin:** the new Finalize row mirrors the PR-Review row (`url.Error` around `ErrTokenRejected`). Its `GH_TOKEN, GITHUB_TOKEN` reason text matches `githubHaltReason`, and `LandingStopPrefix` is checked by the existing assertion.

Two possible findings, both minor:
- **LOW:** `internal/loomcli/landingdeps_integration_test.go` file header, changed text: the new `TestLandingDeps_TaskHasCommitsReadsTheTaskHeadAgainstTheParent`. The header says the file drives "config-change seams ... ConfigChanges ... and Notify". It already left out `TestLandingDeps_PullRequestModeReadsTheParent`, and the new test widens the gap. A reader choosing the file from its header would miss the PullRequest seam tests.
- **LOW:** `internal/fabricengine/parenttips.go`, new exported `HeadHoldsCommitsNotIn`: it has no test in `parenttips_integration_test.go`, though `CommitLog` and `HoldsWork` each have one there. The loomcli test covers it through the closure, including the unresolved-ref error. This only departs from the package's one-test-per-exported-read convention. The file header at line 1 also doesn't mention the new read, although it already omits `CommitLog`.

Further tool calls:
1. `grep -rn "CommitLog("` over the Go files, to find other callers that derive a boolean from `CommitLog`. My context only held the files the review named.
