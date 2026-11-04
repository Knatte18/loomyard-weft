```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out as proposed (F1, F2, F3, F5, F6, and F4's first option) and build the case where each is wrong.

**Findings**

1. **MEDIUM** — `finalize.go` `mergeOpts` (`Squash: fz.deps.Config.Squash || landing.atOnce`), the F1 fix.
   - **Summary:** The F1 fix keys on `landing.atOnce`, which is false on a re-entry where the pull request is already merged, so the parent-side merge there still follows `landing.yaml`'s `squash`.
   - **Case:** A parent has `pull_request=merge` and `squash=false`. GitHub squash-merges, then the fast-forward fails or the run halts, and `mergedButNotPulledReason` says "lym loom resume … lands through the parent-side merge". The re-entry reads `StateMerged`, and `pullRequestLanding` falls through to `return prLanding{}` (`finalize_pr.go:214`), so `atOnce` is false. `mergeOpts.Squash` then comes from `Config.Squash`, which is false. The non-squash merge can create the local merge commit that the protected push refuses, which is the defect F1 describes.
   - **Evidence:** `decision.MergeAtOnce` is known at that point, but only the `StateOpen` and `StateClosed` branches carry it into `prLanding`. The new tests cover only the open and reopen paths, so no row reaches the merged re-entry under a merge parent with `squashOff`.
   - **Possible fix:** Have the merged case also return a landing with `atOnce: decision.MergeAtOnce` and `merge` false. Alternatively, carry the mode on `prLanding` separately from `merge`.

2. **LOW** — `finalize.go` `pushParent(..., landing.atOnce)` and the new `parentMerges` parameter, the F5 fix.
   - **Summary:** `atOnce` is used as a proxy for "the parent already is merge", so it is false on the same merged re-entry.
   - **Case:** On that re-entry a protected-push refusal still advises setting `pull_request=merge`, on a parent that already has it.
   - **Evidence:** This is the F5 no-op advice, left in place on the path above. It is fixed by the same change as finding 1.

3. **LOW** — `finalize_pr.go:206` `if !rec.ReviewEnded && !decision.MergeAtOnce`, the F4 fix.
   - **Summary:** A pull request that a person closed on GitHub on purpose is now silently reopened and squash-merged under a merge parent.
   - **Case:** The record has no ending. Before this change the run halted and offered "lym batten cancel". Now the closed state is read as "reopenable", so a deliberate close cannot stop the landing except through cancel.
   - **Evidence:** Under merge mode `PRReview` writes no ending, so `ReviewEnded` cannot tell a deliberate close from a stray one. The reviewer offered halting with corrected text as the alternative, and the fixer chose the more permissive option without noting this case.

**Further tool calls:** none. The context already held every file and test needed.
