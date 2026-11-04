```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the fix's changed code (fix-code-correctness lens).

**LOW: `atOnce` is lost when the recorded pull request is already merged under a merge parent**
- Changed text: `internal/landingshed/finalize.go:336`, `mergeOpts` with `Squash: fz.deps.Config.Squash || landing.atOnce`. Also the `pushParent(..., landing.atOnce)` calls at lines 342 and 365.
- Input I ran: parent mode `merge`, `landing.yaml` `squash=false`, a recorded pull request whose `Status` is `StateMerged`. This is the re-entry after a `mergedButNotPulledReason` halt.
- Result: `pullRequestLanding` reaches its final `return prLanding{}` at `finalize_pr.go:214`, so `atOnce` is false.
  - The parent-side merge then uses `Config.Squash` (false), which is the F1 defect, because GitHub squashed that pull request.
  - A protected-branch refusal on that push still gets the "set pull_request=merge" advice (F5).
- Why it matters: `atOnce` is only set on the open and reopen paths, but the premise of F1 and F5 is "the parent is `merge`", which also holds for a merged pull request. The merge parent's first pass is fixed; the re-entry after a halt is not.
- Test gap: no row covers a merge parent with a merged record, so nothing pins either behaviour.
- Possible fix: set `atOnce: decision.MergeAtOnce` on the merged and fallthrough returns, or carry the merge-parent flag on `prLanding` separately from `merge`.

I found nothing else. Traced the other changed paths, each against its pre-fix behaviour:
- **F1 row:** merge parent, `squashOff`, open pull request. Before the fix the parent-side merge got `Squash` false, so `wantSquashedMerges` fails. The row pins F1.
- **F2 row:** reopen with `reviewEnded`, merge parent, `squashOff`. Without `atOnce` on the reopen branch, the GitHub merge call reads `|false|` and mismatches `wantGitHubMerge`. The row pins F2.
- **F4 row:** merge parent, closed record with no operator ending. Before the fix it halts, and now it reopens and merges. The row pins F4.
- **F5 row:** merge parent, open pull request, parent push refused as protected. With `parentMerges` false the reason would hold "lym config beam", so `wantNotInReason` fails. The row pins F5.
- **F3 assertions:** `openedRecord` matches what `openAtOnce` writes (Number 7, URL, Base `main`, Head `task-branch`, GatedHead `task-head`, ReviewEnded true). A wrong field fails the `key` comparison.
- **`pushParent` and `protectedParentWayForward`:** I found no caller left on the old signature. Both `finalize.go` calls and both `finalize_test.go` calls were updated.

Further tool calls: none, because my context already held every changed file.
