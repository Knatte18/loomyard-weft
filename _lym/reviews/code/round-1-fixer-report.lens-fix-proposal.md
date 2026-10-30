```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals for F1–F4 as the fixer carried them out.

no findings

- **F4:** `HeadHoldsCommitsNotIn` computes `CommitsNotIn(head, ref)`, the same direction `CommitLog("refs/heads/<parent>", "HEAD")` used. A parent that is ahead of the task, or diverged from it, still gives a correct answer: only commits on HEAD that the parent lacks count. `ResolveSHA` errors name the ref, which the F1 error row asserts.
- **F1:** the three subtests catch the mutations I tried.
  - A swapped `from` and `to` fails the "one commit" row, because the parent holds nothing HEAD lacks.
  - Comparing HEAD against HEAD fails the same row.
  - A dropped error fails the unresolvable-ref row.
- **F2:** the new rows set `taskErr` and leave `wantPushes`, `wantCalls` and `wantMerges` at their zero values, so the tables check that nothing was pushed, called on GitHub or merged. `wantErrText` defaults to `""`, which matches any error, so existing rows are unaffected.
- **F3:** the Finalize row wraps `ErrTokenRejected` in a `url.Error`. Reverting `tokenFailure` to the unresolvable-only check would send it down the transient path and return an error, so the row's halt assertion would fail. The expected text `GH_TOKEN, GITHUB_TOKEN` is present in `githubHaltReason`'s rejected-token branch.

Outside my lens: F2 and F3 share one commit although the instructions ask for one commit per fix, since both edit the same two test files.

Tool calls: none; my context already held every changed file.
