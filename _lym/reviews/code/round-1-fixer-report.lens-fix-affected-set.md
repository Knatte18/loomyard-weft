```yaml
lens: fix-affected-set
outcome: reported
```

Task: affected-set check of this round's fix change set.

Findings left unchecked: none.

1. **MEDIUM**: `finalize.go` `gateUnlocked`, `!fz.needsGitHubOnDisk()` (F4). One kind of GitHub landing still merges the owed tips in the lock-free pass.
   - `needsGitHubOnDisk` answers false for mode `off` (and for a beam only when it has no recorded PR and the mode itself does not demand one).
   - Under the lock, `resolveWithParentRule` (`pullrequestmode.go`) can still turn that into a GitHub merge, because GitHub's rules require a pull request (`case parentRuleRequired: wanted, demand = true, ...`).
   - With an owed parent, `gateUnlocked` then merges the local owed tips, and `landUnderLock`'s `landing.merge` → `parentPushBlocks` ends the call. That is F4's stated sequence.
   - The new wording ("A landing known to need GitHub") matches the code. The finding's premise is only partly resolved, and closing it fully would need a GitHub read before the lock, so the fixer-report should at least state this residue.

2. **MEDIUM**: `finalize.go` `gateUnlocked`, `!fz.needsGitHubOnDisk()` (F4, same set as F2). This is a third upstream fallback of an ahead parent in the lock-free pass, and no test fails if it breaks.
   - The new test `TestFinalize_LockFreePassMergesTheUpstreamTipOfAnAheadParent` covers owed false, nil seam and the second read, with no pull-request wiring.
   - The suite passed both before and after the F4 edit, so reverting the argument to `true` would go unseen.
   - A row with a trail mode or a recorded pull request, owed true, asserting first tip `u0`, would pin it.

3. **MEDIUM**: `doc.go`, "A landing known to need GitHub always merges the upstream tip there." This is false under the push skip or with no upstream.
   - `parentMergeTip` skips the whole relation switch when `deps.PushSkipped` is set, and falls through on `ParentNoUpstream`, returning `tips.Local` whatever `takeOwedLocal` says.
   - `needsGitHubOnDisk` can be true there (recorded PR or trail mode).
   - Dropping "always" and scoping the sentence to the owed-ahead case fixes it. The `gateUnlocked` doc line reads correctly, since it is framed as an exception to the owed-ahead sentence.

4. **LOW**: `parentalign.go`, second `readAheadByOwed()` under the lock (F1). Its error return has no test. The existing "failed read of whether the parent is ahead by owed landings alone" row fails the first read only.

The other sets are covered:
- **F1:** `parentMergeTip` is the only site that reads the local tip after an unlocked owed check.
- **F3:** Parent-Align has one call, and it is pinned.
- **F5:** `docs/overview.md` (the `Parent-Align` and `Finalize` bullets) and the `Finalize.Call` doc do not claim "never the local tip", so none is stale.

Tool calls:
1. Repo-wide grep for wording about the local and upstream tips and for the seam names (F5 set); my context held only `internal/` matches for symbol names. The output was truncated.
2. Grep of the landingshed tests for lost-race, push-state and GitHub wiring (F2/F4 test set), which my context lacked.
3. Read `pullrequestmode.go` and grepped `pullRequestLanding` (F4's set of GitHub landings); neither file was in context.
4. Narrower grep of docs and sources for owed-tip and lock-free wording, because call 1 was truncated.
5. Read the `Finalize.Call` doc and the `docs/overview.md` Finalize bullet, which were outside the spans in context.
