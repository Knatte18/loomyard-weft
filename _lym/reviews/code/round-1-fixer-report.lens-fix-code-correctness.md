```yaml
lens: fix-code-correctness
outcome: reported
```

Lens fix-code-correctness: I ran each changed function and each added test through concrete inputs.

**LOW: `PairTaskBranchOrOrigin` doc overclaims, in `internal/fabricengine/pairbranch.go`.**
- The doc says "It asks origin only when nothing local names the pair, so a pair with any local remnant makes no network call."
- The first half is accurate: `pairTaskBranchLocal` returns `found=false` whenever no local branch or worktree branch matches the pair's names.
- The "so" clause is not. Take a task worktree checked out on `elsewhere`, with no local bare or prefixed branch and no records branch. The worktree is a local remnant, the local read returns `found=false`, and `originPairBranch` runs.
- The code is right in that case, since the pair really is unnamed locally. Only the second clause of the doc is wrong. Drop it, or say "no local branch or worktree names the pair".

No other defect found:
- **`pairTaskBranchLocal`:**
  - The inner `branch, err :=` shadowing is safe, because every return passes values explicitly.
  - An unresolvable records root now skips to the local-branch step, which is what F6 asked.
  - A failed `worktreeBranchAt` is still returned.
- **`PairTaskBranchOrOrigin`:**
  - A local error is returned as before.
  - An origin lookup error returns `"", err`. Both callers log it and carry on.
  - With no origin remote it returns the current prefix's name.
- **`removePair` and `refuseRemoval`:**
  - `RemoveRefusal` keeps its signature.
  - `refuseWhenNothingRemains` has a single caller, which was updated.
  - A non-empty `onOrigin` implies nothing was local at the refusal, so the `localPartRemains` guard in `removePair` only matters if a local part appeared between the two reads.
  - A launcher-only pair now makes no origin call, and an origin-only pair is asked once per `Remove`.
  - `RemoveLocal` ignores `onOrigin`, so its behaviour is unchanged.
- **Tests:** each added test fails with its defect in place.
  - **Defect in `PairTaskBranch`:**
    - Without the unrelated-branch check, the `elsewhere` row returns `elsewhere`.
    - Without the `weftBranchExists` clause, the records-branch-only row returns `hanf/slug`.
    - Without the records step in `originPairBranch`, the records-only-on-origin row gets `ErrPairNotFound`.
  - **Slug and branch derivation under a prefix:**
    - The `t4` row of `TestTaskBranchSlugs` needs the trimmed slug.
    - The `--parent` bare-slug test needs both names accepted.
    - The `battencli` and `pairteardown` tests would see `hanf/&lt;slug&gt;` under the old `cfg.BranchPrefix + slug`.
  - **Gap, no `Remove` test for F4:** none counts origin calls, which the review did not require.
  - **Gap, no test for F6:** it needs `RecordsRepoRoot` to fail while `PrimeName`'s `List(l.AnchorPath())` call fails, and the earlier steps' `List` calls succeed, so no row can reach it.
- **Not a regression:** `mergeSourceInFlight(l, pair.warpBranch)` in `refuseRemoval` still uses the current prefix's name for an origin-only bare-slug pair, though `onOrigin` is now known. This predates the diff and `onOrigin` is not used there.

Tool calls: none; my context held every changed file and the gate output.
