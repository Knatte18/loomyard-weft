```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens — for each changed signature, output or behaviour, what else reads it and whether the diff left that wrong.

**Per-change set**

- **`ownedManagedBranch(l)` and `resolveManagedBranch`: prefix dropped.**
  - Every caller is updated: `checkout.go`, `reconcile.go`, `cleanup.go` (two sites), `weftwiring.go`, `export_test.go` and `destroy_test.go`. Covered.
  - The helper comment in `destructivegaps_integration_test.go` is stale. Finding 1.
- **`removeWeftWorktree` and `deleteWeftBranch(OnRemote)`: prefix parameter dropped.** `add.go`, `remove.go`, `weftwiring.go`, `cleanup.go` and `remoteteardown.go` all match. Covered.
- **`remoteTeardownJob.BranchPrefix` removed.**
  - The construction sites in `remove.go`, `cleanup.go` and `taskbranchdiscard.go` are updated, and `remoteteardown_integration_test.go` drops the key.
  - `RunTeardownChild` uses plain `json.Unmarshal`, so a job carrying the old key still parses. Covered.
  - The field comment is misplaced. Finding 3.
- **`slugOfBranch` and `pairBranchNames`.**
  - `cleanup.go`, `cleanupremotewarp.go` (two sites), `pairparts.go` (`TaskBranchSlugs`) and `addparent.go` are switched. Covered.
  - `TestTaskBranchSlugs` uses an empty prefix, and the `cleanupremotewarp` test's "unmanaged" row doesn't name a prefix reason. So the dropped "lacks the prefix" refusal asserts nothing stale. `grep` finds no assertion on it and no doc left stating it.
  - The remaining `cfg.BranchPrefix + slug` uses are `add.go:194`, which names a new pair, and `battencli/wire.go:1065`, which names the branch Add just failed on. Both are legitimate.
- **New `Topology.PairTaskBranch`.** Its callers are `remove.go`, `taskbranchdiscard.go`, `pairteardown/pullrequest.go` and `battencli` (`readopt.go`, `wire.go`). Its name carries no fabric side, which fits `PATTERN-fabric-vocabulary`. `PATTERN.md` and `pattern/` declare no changes, and the branch carries none. Covered.
- **`RemoveResult.SiblingBranch`.** `fabriccli/fabric.go` reads it. The CLI envelope builds a `fields` map (`fabric.go:1154`), so the field is not emitted and `removefields_test.go` and `docs/overview.md` need no update. Covered.
- **`branch_prefix` semantics documented.**
  - `fabricengine/doc.go` is updated.
  - The `branch_prefix` description in `template.yaml` is not. Finding 2.
  - `SANDBOX-FABRIC-SUITE.md` lines 131 and 560 describe create-time naming only. Covered.
- **Commit messages.** None of the four carries a `Pattern-Departure:` trailer. I found no claim the diff fails to bear out.

**Findings**

1. LOW, consistency: `internal/fabricengine/destructivegaps_integration_test.go:721` still says the helper gates "via ownedManagedBranch(l, "")". That signature no longer exists, because the diff changed the call at line 729 but not this comment.
2. LOW, consistency: `internal/fabricengine/template.yaml:1` reads "prefix prepended to the slug to form the branch name … empty = branch == slug". The decision record says `branch_prefix` now names only the branches of pairs created after the change. The template is the config's user-facing description, and it doesn't say that or that the prefix can change with pairs live.
3. NIT, consistency: `internal/fabricengine/remoteteardown.go:62-68` has one comment reading "WeftBranch and WarpBranch are the pair's branches as the command read them…" above `WeftBranch` only. `WarpBranch` is declared six lines later, after `DeleteWeftOnOrigin`, with no comment. Each field should sit beside its own comment.
4. LOW, design: the local-remnant test in `removePair` (`remove.go:207`) leaves out `launchers`, but `refuseWhenNothingRemains` (`remove.go:563`) counts it as a local part. A pair with only a launcher directory left therefore makes an `ls-remote` call in `removePair`, though the refusal probe made none. That contradicts the doc claim "a pair with any local remnant never costs a network call" (`remove.go:560`). When the bare-slug pair sits only on origin, an origin-only pair also costs two origin lookups, one in each place.
5. LOW, design: `pairteardown/pullrequest.go:45` takes `PairTaskBranch` after the worktree is gone. For a bare-slug pair with no local branch left, it returns `names[0]` (current prefix plus slug) rather than the real branch. The name feeds only the warning in `closeOpenPullRequest`, which calls nothing when `gone` is true. So the effect is a wrong branch name in a log line, not a missed pull request.

**Tool calls (3 of 6)**
1. `grep` for "lacks the branch prefix", "not fabric-managed" and `TaskBranchSlugs` across `.go`, `.md` and `.yaml`: to find asserts and docs of the dropped refusal and the changed slug listing, which my context lacked.
2. `grep` for `code_branch_kept_reason` and `"sibling_branch"` across the repo, plus a `grep` of the sandbox suite for the prefix: to find consumers of the Remove result fields and docs that describe `branch_prefix`.
3. `grep` for prefix use in `destructivegaps_integration_test.go` and in the `pairteardown` tests: to see whether any ownership or PR-lookup test depends on the old prefix behaviour.
4. `sed` of `pairparts_integration_test.go`, `destructivegaps_integration_test.go` and `cleanupremotewarp_integration_test.go`: to read the `TestTaskBranchSlugs` rows, the stale comment and the "unmanaged" row.
