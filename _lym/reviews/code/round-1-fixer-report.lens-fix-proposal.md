```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fix carried out (F1–F5), looking for cases where they are wrong or break something.

**Findings**

1. **LOW.** Changed text: `finalize.go`, `gateUnlocked` (`parentMergeTip(..., !fz.needsGitHubOnDisk())`) and its new doc sentence "A landing known to need GitHub is the exception". Also the `doc.go` sentence added after line 42.
   - **Summary:** F4's proposal does not cover a landing that needs GitHub only because of GitHub's repository rules.
   - **Evidence:** `needsGitHubOnDisk` (`finalize.go:609-642`) returns true only for a recorded pull request, a beam record, trail mode, or review mode. It passes `parentRuleRequired=false` to `ResolvePullRequest`.
   - `ResolvePullRequest` (`pullrequestmode.go`, `case parentRuleRequired`) opens a pull request under mode `off`, or on a beam under review, when the rules require one.
   - **Concrete case:** mode `off`, no recorded pull request, rules requiring one (added after an earlier local landing left tips owed). The lock-free pass merges the owed tips, then `landUnderLock` ends in `parentPushBlocks`. That is the F4 scenario.
   - The case is narrow, and "known to need GitHub" stays literally true. Closing it would need a rules read with no lock, so it fits better as a stated limit than as a code change.

**No breakage found in the other proposals**
- **F1:** `AheadByOnly` answers true for a level branch, so an owed push completing before the lock still yields the local tip and wastes no race. A plain commit arriving before the re-check yields the upstream tip.
- **F2 and F3:** the new tests fit their cases.
- **F4:** apart from the finding above, a landing that `needsGitHubOnDisk` flags while the parent owes a push ends at `finalize.go:423` before the head comparison. So merging the upstream tip there causes no lost-race loop when `ParentPushState` is wired.
- **F5:** the rewritten `parentMergeTip` doc matches the code.

**Outside this lens:** no test fails if `!fz.needsGitHubOnDisk()` at the `gateUnlocked` call is replaced by `true`, so the F4 behaviour is unpinned.

**Tool calls**
1. Read `ResolvePullRequest` in `internal/landingshed/pullrequestmode.go`. My context did not show how rule-required pull requests are resolved.
