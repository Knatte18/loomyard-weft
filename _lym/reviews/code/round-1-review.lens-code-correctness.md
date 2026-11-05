```yaml
lens: code-correctness
outcome: reported
```

**code-correctness list.** Each line: function or branch, the case I ran, verdict.

- `pairBranchNames`: prefix `""` gives `[slug]`; prefix `hanf/` gives `[hanf/slug, slug]`. Correct.
- `slugOfBranch`, empty prefix: `TrimPrefix` returns the branch unchanged. Correct.
- `slugOfBranch`, prefix `hanf/`, branch `hanf/x`: gives `x`. Correct.
- `slugOfBranch`, prefix `hanf/`, branch `x`: gives `x`. Correct.
- `slugOfBranch`, prefix `hanf/`, branch `main`: gives `main`. See finding 1.
- `PairTaskBranch`, task worktree on `slug` under prefix `hanf/`: returns `slug`. Correct.
- `PairTaskBranch`, task worktree on a branch outside the names: falls through to the later steps. Correct.
- `PairTaskBranch`, detached HEAD: `worktreeBranchAt` returns `""`, which is not in `names`, so it falls through. Correct.
- `PairTaskBranch`, task worktree gone and records worktree on `slug-weft`: returns `slug`. Correct.
- `PairTaskBranch`, only a local `slug` branch left: the loop returns `slug`. Correct.
- `PairTaskBranch`, nothing left at all: returns `names[0]`. Correct.
- `PairTaskBranch`, both `hanf/slug` and bare `slug` present locally: returns the prefixed one. Acceptable.
- `PairTaskBranch`, `List` or `RecordsRepoRoot` failure: the error is returned. See finding 2.
- `PairTaskBranch`, origin-only bare-slug pair with prefix set: returns `hanf/slug`, which is wrong. See finding 3.
- `worktreeBranchAt`: skips `Main` and `Prunable` entries, and uses the same path comparison as `isRegisteredLinkedWorktree`. Correct.
- `inspectPair`: reads `warpBranch` once, then derives `taskBranch` and `siblingBranch` from it. Correct.
- `removePair`, origin-only pair, not local mode: `originPairBranch` sets `pair.warpBranch`, and every later step uses `warpBranch` and `weftBranch`. Correct.
- `removePair`, origin lookup returns `""`: keeps `names[0]`. Correct.
- `removePair`, local mode with nothing local: origin is not asked. Correct. `RemoveRefusal` still asks origin in that case, as before.
- `refuseWhenNothingRemains`, any local part present: no network call. Correct.
- `refuseWhenNothingRemains`, nothing local, either name on origin: returns nil. Correct.
- `RemoveRefusal` and `removePair` agreement: both call `inspectPair`. Origin is queried twice for an origin-only pair, once in the refusal and once in `removePair`. This is only a cost, not a defect.
- `RemoveResult.SiblingBranch`: set to `weftBranch` even when no sibling existed. The field comment says "tore down". It is only read when `RemoteBranchError != ""`, so the effect is harmless.
- `DiscardTaskBranch`, local branch present: uses it. Correct.
- `DiscardTaskBranch`, no local branch, bare name on origin under prefix `hanf/`: `originTaskBranch` returns the bare name, and the job carries it. Correct.
- `DiscardTaskBranch`, no local branch and none on origin: the job runs `remoteHeadTip` on `names[0]`, gets `""`, and does nothing. Correct.
- `remoteTeardownJob`, a job file written with the old `branch_prefix` key: `json.Unmarshal` ignores the unknown key. Correct.
- `runRemoteTeardown`: uses only `job.WeftBranch` and `job.WarpBranch`. Correct. Cleanup jobs leave `WarpBranch` empty but never use it.
- `ownedManagedBranch` and `resolveManagedBranch`: accept only names `WeftWarpSlug` accepts. Every caller deletes a `-weft` records branch, so nothing legitimate is refused. Correct.
- `resolveAddParent`, prefix `""`: names are `[parent]`. Correct.
- `resolveAddParent`, bare checkout under prefix `hanf/`: accepted, and `recordsBranch` is built from the actual branch. Correct.
- `resolveAddParent` error text: names `pairNames[0]`. Correct.
- `keptReason` in `cleanupremotewarp.go`: see finding 1.
- `TaskBranchOnOrigin`: no origin gives `false`, nil; either name on origin gives `true`. Correct.
- `battencli` `hubTaskBranch` and `TaskBranchGone`, bare local branch present: `PairTaskBranch` returns it, so `gone` is false. Correct.
- `battencli` `hubTaskBranch` and `TaskBranchGone`, bare branch only on origin with prefix set: `TaskBranch` returns `hanf/slug`, which is wrong. See finding 3.
- `wire.go:1032`: `PairTaskBranch` runs only when the worktree is present, and its error is propagated. Correct.
- `wire.go:1065`: `cfg.BranchPrefix+slug` after `Add` is correct, since `Add` names the new pair's branch.
- `add.go:194`: same reasoning. Correct.
- `closePairPullRequest`, gone branch: `PairTaskBranch` failure is logged and the branch name stays `""`. Correct.
- `closePairPullRequest`, gone branch: `closeOpenPullRequest` with `gone` only logs a warning and calls no GitHub API. See finding 3.

**Findings**

1. **LOW, `design`.** `slugOfBranch` classes any branch lacking the prefix as a bare slug.
   - `internal/fabricengine/pairbranch.go:25-27`, `pairparts.go:74`, `cleanupremotewarp.go:197`.
   - With prefix `hanf/`, `TaskBranchSlugs` now returns `main`, `develop` and other non-pair branches as slugs. Before, `CutPrefix` skipped them.
   - Its caller `adoptcli/fixloop.go:303` numbers new fix slugs past the returned slugs, so a non-pair `fix-N` branch now reserves a number.
   - `keptReason` also reads a non-prefixed branch as managed whenever `<branch>-weft` or an `archive/<branch>/*` tag exists on the weft origin.
   - The destructive gate and the landed-work check still bound the damage.

2. **LOW, `design`.** `PairTaskBranch` can make a half-torn pair unremovable.
   - `internal/fabricengine/pairbranch.go:44-48`, called from `inspectPair` at `remove.go:545`.
   - A records worktree directory is present but `RecordsRepoRoot` fails. `inspectPair` then errors, so `RemoveRefusal`, `Remove`, `PairParts` and `DiscardTaskBranch` all fail. Before the diff, `inspectPair` never called `RecordsRepoRoot`.
   - `removeSibling` at `remove.go:493-501` tolerated this failure, which the diff's new early error now pre-empts.

3. **LOW, `design`.** `PairTaskBranch` never consults origin, so a bare-slug pair with only its origin branch left is named `hanf/slug` once the prefix is set.
   - `internal/fabricengine/pairbranch.go:66`.
   - In `closePairPullRequest` this affects only the warning's `branch` field (`pullrequest.go:84`). `closeOpenPullRequest` calls no GitHub API when `gone`, so no PR is found or closed either way.
   - In `battencli/readoptstop.go:206-210` the same wrong name feeds `RecordsBranchName(taskBranch)` and `BranchFile(recordsBranch, …)`. For an origin-only bare pair with a prefix set, the re-adoption looks up `hanf/slug-weft` instead of `slug-weft`.
   - `TaskBranchGone` itself is correct, because it uses `TaskBranchOnOrigin` under both names.

4. **NIT, `consistency`.** The comment above `WeftBranch` in `remoteTeardownJob` also describes `WarpBranch`, which is declared several fields later.
   - `internal/fabricengine/remoteteardown.go:62-68`.
   - The struct's field grouping and gofmt alignment are now split.

Tool calls:
1. `grep` for `TaskBranch(` callers in `battencli/readopt*.go` and for `TaskBranchSlugs(` callers. The manifest lacked the consumers of those two functions.
2. Read `readoptstop.go` around line 206. The manifest lacked how the re-adoption uses the returned name.
