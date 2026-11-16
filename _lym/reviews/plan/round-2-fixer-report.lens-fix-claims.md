```yaml
lens: fix-claims
outcome: reported
```

Fix-claims lens: checked each sentence the fix added or rewrote against the code at HEAD.

**Finding**

- **MEDIUM**, `_lym/plan/06-run-hand-over.md`, `HandOver` step 2, the added sentence "The rejected pair push fetched only the pair's records branch, so without this fetch an intent already on origin could be named dropped and one not on origin missed."
  - **Summary:** the code does not support this claim. The fetch that `retryWeftPush` runs is a whole-remote `git fetch`, not a fetch of one branch.
  - **Evidence:** `internal/fabricengine/pushclassified.go:599` calls `repo.Fetch()`. `internal/gitrepo/pull.go:35-39` runs a bare `r.run("fetch")`, which updates every remote-tracking ref of the weft repository's default refspec.
  - **Shared refs:** linked worktrees of one repository share their refs. If the pair's records worktree and the prime's are worktrees of one weft repository, this fetch already refreshed the prime records branch's tracking ref.
  - **Not settled:** I did not confirm two points within my call budget. First, whether those records worktrees are worktrees of one weft repository. Second, whether `withRemoteRefs` narrows the refspec.
  - **Partial timing exception:** card 5 reads the takeover before any replay, so on that path `retryWeftPush` may not have run yet. That leaves the claim partly right about timing.
  - **Suggested change:** reword the reason so it does not rest on what the rejected push fetched. For example: "so the read does not depend on whether, or when, an earlier fetch refreshed the prime records branch."

**Claims that hold**

- `CommitAnchoredPaths` writes no trailer: `commitweftpaths.go:41-82`.
- `CommitAnchoredPathsWithTrailers` with nil trailers writes `Warp-SHA: &lt;warp HEAD&gt;`: `trailercommit.go:78-116`. It leaves the line out only while the warp has no commit, which does not affect a run's transition.
  - `Open(l)` is `newPaired(l.WorktreePath(), RecordsWorktree(l))` (`open.go:9`), so it resolves for the prime location too.
- `updateRemoteBranch` runs `checkPathRequest`, which checks containment, ownership and dirtiness in that order, with force consulted only in the dirtiness step: `destroy.go:858-887`, `:1627`.
  - It takes `remoteBranchUpdateRequest{pathReq, …, leaseSHA}`: `destroy.go:212-227`.
  - `dirtinessNA` needs a non-empty reason, which the fix gives: `destroy.go:865`, `:473`.
- `ownedWarpCheckout` counts any worktree of the repository, the prime included: `destroy.go:275-279`.
- `git reset --keep` keeps local changes and aborts where a changed file differs between HEAD and the target. That matches git's documented behaviour.
- "a landing push clears the record, every run with it": `ClearPushFailure` removes the whole file for each side and branch (`failedpush.go:200-205`).
- Readopt's "the run already halts at the failed stop" holds. `readoptstop.go:98-110` logs a `CommitFailed` error and returns `stop` whatever happens, so any push error `CommitFailed` returns only gets logged by its caller.

**Tool calls**

1. grep of `resetHardTo`/`ownedWarpCheckout` callers and `container:` values: to check the "contained in the hub" and prime-ownership claims, which my context did not hold.
2. grep of `ClearPushFailure` plus `readoptstop.go:85-115`: to check the landing-clear and failed-stop-halt claims; my context had neither.
3. `Open` and `ClearPushFailure` bodies: to check that the trailer commit resolves at a prime location and how a record is cleared; my context lacked both.
4. grep for fetch in `retryWeftPush`: to check the claim about what the rejected push fetched.
5. `gitrepo.Fetch` body and `retryWeftPush` header: to see what that fetch covers.
