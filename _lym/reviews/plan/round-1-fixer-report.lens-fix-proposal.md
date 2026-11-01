```yaml
lens: fix-proposal
outcome: reported
```

fix-proposal lens: for each finding fixed as proposed, I looked for a concrete case where the proposal is wrong.

Proposals checked: F1, F2, F3 (the parent's decision carried on card 13), F4, F5, F6, F8, F9, F10, F11. F7 was fixed through its second option, the accepted bound, so its claim belongs to fix-claims.

**MEDIUM — card 11, `reconcileWarnings` (F2's proposal)**
- **Changed text:** "`<branch> is <state> against origin and was left unmoved: run lym fabric pull in <worktree>, then re-run lym adopt with the same arguments` … so every line names `lym fabric pull` as D2 requires", plus the TestAdopt row "a diverged branch of an adopted beam's pair carrying a `WayForward` is a warning naming `lym fabric pull`".
- **Summary:** For two of the three states the warning covers, the first step it names does nothing or refuses, so a re-run of adopt meets the same warning again.
- **Case 1, `ahead`:** `BranchAhead` means "origin's head is an ancestor of the local tip" (`internal/fabricengine/originreconcile.go:130`). There is nothing to pull, so `lym fabric pull` leaves the branch as it is, and the re-run of `lym adopt` emits the same warning.
- **Case 2, `diverged` code branch:** `Fabric.Pull` refuses with `ErrWarpDivergedUnpushed`, "warp remote diverged and local warp has unpushed commits; aborting, no changes" (`internal/fabricengine/pull.go:111`, raised at `pull.go:687`). A diverged code branch is that state, so the named step aborts, and only the parenthesised `WayForward` (`git merge` and push, `failedpush.go:258-266`) settles it.
- **Why the proposal is wrong:** The old text, which named the reconcile's own `WayForward` first, could be carried out. D2's wording is the premise behind the proposal, so a sound fix keeps `lym fabric pull` only for `rewritten`. For `diverged` and `ahead` it leads with the reconcile's `WayForward` and keeps `lym fabric pull` as the fallback. That departure needs the parent's word through `lym creel post --parent`, since D2 names `lym fabric pull`.

No other finding: F1's `endOnlyError`, F3's vocabulary gate, F4's resume texts, F8's starting case, F9's `--keep-local` row, F10's re-signs, F5, F6 and F11 each hold up against the cases I built.

Tool calls:
- One grep of `fabricengine/pull*.go` and `originreconcile.go` for how pull treats ahead and diverged branches. My context held the reconcile's way-forward code but not `Fabric.Pull`'s behaviour.
