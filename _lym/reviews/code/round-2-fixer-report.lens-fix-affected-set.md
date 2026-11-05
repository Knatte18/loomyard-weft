```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran the set searches for F1, F2 and F3. F4 names no set.

**Finding**
- **MEDIUM** — `internal/fabricengine/addparent.go:48,66` is a site F2 missed. It reads a pair's branch by name alone.
  - `pairBranchNames(cfg, parent)` gives the current prefix's name and the bare slug.
  - `slices.Contains(pairNames, branch)` then refuses a parent whose worktree has checked out a branch from an earlier non-empty prefix.
  - Take a parent made under `earlier/` with the prefix now `hanf/`. `Add --parent` reports "has `earlier/<slug>` checked out, not its own pair branch" and fails.
  - That is the same fault F2 and decision #692 describe, and `PairTaskBranch` no longer has it. The accepted rule was "a task worktree's branch is its pair's branch when it is a name of the pair or has a records branch".
  - `addparent.go` also takes `pairBranch := pairNames[0]` for its checkout hint, which is fine.
  - The parent worktree is read directly here, so applying the `weftBranchExists(RecordsBranchName(branch))` condition would fix it without calling `PairTaskBranch`.

**Searched with no gaps**
- **F1:** the literal `"(detached)"` now appears only at the `detachedBranch` definition in `worktreelist.go:17` outside tests. The fixture in `matchparent_test.go` stays, as the review said.
- **F3:** `localPartRemains` has no remaining references. `refuseWhenNothingRemains` has one caller, `refuseRemoval` (`remove.go:699`), which feeds the `onOrigin` that `removePair` adopts. `branchPartRemains` is used only inside `refuseWhenNothingRemains`.
- **F2:** besides `pairbranch.go` and `addparent.go`, the only other name-based sites are `originTaskBranch` and `originPairBranch` (`pairbranch.go:122,136`). Those ask origin when nothing local is left, so no worktree exists to read there. `shedverbs/pause.go:51` is unrelated.

**Tool calls**
1. One `Bash` with three greps, covering the F1, F3 and F2 sets. My context held no repo-wide search results.
2. A `Read` of `internal/fabricengine/addparent.go` lines 35–75. The `pairBranchNames` grep named that file as a caller, and my context did not hold it.
