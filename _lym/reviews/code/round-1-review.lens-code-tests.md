```yaml
lens: code-tests
outcome: reported
```

Task: review the diff's tests (code-tests lens) and list, for each new or changed behaviour, the test row that would fail if it broke.

**Behaviour → test that pins it**

1. A `RefusalOther` merge refusal reads the merge state, waits while mergeability is `unknown`, then retries and merges once it reads `clean`. Pinned by `ANotMergeableRefusalWhileGitHubComputesMergeabilityWaitsItOutAndMerges` (`internal/landingshed/finalize_pr_test.go:306-311`). Its expected call list and Done outcome are literals.
2. A reading that says the pull request cannot merge halts at once, with no retry, and the task-branch halt names `GitHub reports: <causes>` plus the cancel. Pinned by `AnotherRefusalHaltsNamingGitHubsMessageTheCauseGitHubReportsAndTheCancel` (`finalize_pr_test.go:299-305`). The call list holds one merge and one reading, so a retry would fail it.
3. The beam-branch halt carries the reported causes. Pinned by `ABeamsOtherRefusalHaltsNamingTheStatusVerbAndNotACancel` (`finalize_pr_test.go:414-420`), which checks `GitHub reports: blocked`.
4. A mergeability that never resolves halts after one minute of polling and names the unknown cause. Pinned by `AMergeabilityGitHubNeverComputesHaltsNamingItAfterTheWindow` (`finalize_pr_test.go:312-319`). It expects exactly 7 Status reads, from literals.
5. When the state can merge but the refusal outlasts the window, the loop retries a bounded number of times and halts naming only the resume. Pinned by `ARefusalThatOutlastsTheWindowWhileGitHubReportsItCanMergeHaltsNamingOnlyTheResume` (`finalize_pr_test.go:320-328`). It expects 7 merge+reading pairs, and `wantNotInReason` excludes the cancel and "approves". This settles the focus question about unbounded growth for the task branch.
6. A pending refusal still reads no merge state. Pinned by the pending-window row at `finalize_pr_test.go:292` and the beam row at `:412`, whose call lists hold only merges.
7. A method refusal still reads no merge state. Pinned by the row at `finalize_pr_test.go:297` (`wantCalls` `Status 7`, merge).
8. The head-mismatch cause now reads "the expected head". Pinned by `TestReadMergeState/HeadMismatchNamesBothHeads` (`internal/pullrequest/mergestate_test.go:75`). No beamcli test asserts the old or the new text (grep of `internal/beamcli/*_test.go` found none).
9. A `ReadMergeState` failure inside the merge loop goes to `githubFailure` with "read whether the pull request can merge". **No test.**
10. A beam whose reading says it can merge, past the window, gets the resume-only text, which returns before the beam branch and drops `lym beam pr status`. **No test.**

**Findings**

1. **MEDIUM**, class `design`, at `internal/landingshed/finalize_pr.go:503-506`. No row reaches the error path the diff adds when `ReadMergeState` fails after a refusal. A row would script a Status error after a `RefusalOther` merge and check the "read whether the pull request can merge" halt or error.
   - Evidence: the fake's `RequiredChecks` never fails (`internal/landingshed/prreview_test.go:123-125`).
   - Every new row scripts only successful Status reads (`finalize_pr_test.go:299-328`, `:414-420`).
   - The mutation report counts "1 uncovered changed blocks", which fits this branch.
   - The same branch also gets a failed sleep inside `ReadMergeState`, such as a cancelled context, and that path is uncovered too.
2. **LOW**, class `design`, at `internal/landingshed/finalize_pr.go:577-580`. No beam row covers a reading that can merge past the window. In that case `mergeRefusalReason` returns the resume-only text before its beam branch, so the beam halt no longer names `lym beam pr status <slug>`. No test pins this outcome, whether it is intended or not.
   - Evidence: the only beam other-refusal row scripts `openAt("blocked")` (`finalize_pr_test.go:416`).
   - Outside my lens: the doc says a beam's refused merge "names GitHub's message, `lym beam pr status <slug>` and `lym loom resume`" (`internal/landingshed/doc.go:632`). The correctness and affected-set forks should check that claim against this path.

Tool calls:
- grep of the landingshed and beamcli tests for an error-path row, the fake's `RequiredChecks`, and the old/new head text. Context lacked the fake's methods and the beamcli test assertions.
- `sed -n 33,80p internal/landingshed/prreview_test.go`: how the scripted fake repeats its last result, which confirms the expected call counts. Context lacked the fake's definition.
