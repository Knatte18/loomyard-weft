# Card 3 — add-remote-preflight

**Edit:**
- `internal/fabricengine/add.go`

**Create:**
- `internal/fabricengine/remoteleftover.go`
- `plan:internal/fabricengine#ErrRemoteLeftover` -> `type ErrRemoteLeftover struct`
- `plan:internal/fabricengine#weftLeftover` -> `type weftLeftover struct`
- `plan:internal/fabricengine#probeWeftLeftover` -> `func probeWeftLeftover(l *lyxcwd.Location, slug, weftBranch string, localExists bool) (weftLeftover, error)`
- `internal/fabricengine/add_remoteleftover_integration_test.go`

**Uses:**
- `internal/fabricengine#resolveArchiveTip`
- `internal/fabricengine#weftBranchExists`
- `internal/fabricengine#WeftRepoRoot`
- `internal/fabricengine#originRemoteName`
- `internal/gitrepo#Repo.IsAncestor`

**Intent:**
Give `Topology.Add` pre-flight probes of both origins, so a leftover remote branch from a removed pair is either proven replaceable or refused before the first mutation, never rejected mid-Add at a push.
The probes are read-only git through `gitexec`, so they live in a new `remoteleftover.go`, not `destroy.go`.

`remoteleftover.go` holds:
- `ErrRemoteLeftover`, the one typed refusal for all three outcomes below, with exported `Slug`, `Branch` and `RemoteTip` fields and an unexported closed-enum kind.
  `Error()` names the branch, the remote tip and the remedies for its kind:
  an unarchived weft leftover names `lyx fabric remove <slug>` on the hub that holds a live pair (it archives the branch), `git push origin --delete <branch>` once no hub holds the pair and its content is disposable, and a different slug;
  a diverged adopted weft branch names the same hand deletion and a different slug;
  a diverged warp leftover names `git push origin --delete <branch>` once its work is landed, and a different slug.
- `weftLeftover`, the probe's answer: the remote tip and the covering archive tag, both empty when there is nothing to replace.
- `probeWeftLeftover(l, slug, weftBranch, localExists)`, run in the weft repo against `originRemoteName`:
  `ls-remote --heads` for the branch, returning the zero answer when it is absent.
  With `localExists` (today's adopt path) it returns the zero answer when the remote tip equals or is an ancestor of the local weft tip, and an `ErrRemoteLeftover` otherwise.
  Without it, it lists `refs/tags/archive/<slug>/*` with `ls-remote --tags`, preferring a `^{}` peeled line when one is present;
  a tag whose target equals the tip proves it outright, and otherwise it fetches the remote branch tip and the listed tags into `FETCH_HEAD` with `git fetch --no-tags --refmap= origin <refs>...`, so the fetch creates no local branch, remote-tracking ref or tag, then tests each tag target with the ancestor check.
  A covering tag yields `weftLeftover{tip, tag}`; none yields an `ErrRemoteLeftover`.
- `probeWarpLeftover(l, warpBranch)`, run in the warp worktree: `ls-remote --heads origin` for the branch, nil when absent or when its tip equals or is an ancestor of the warp `HEAD` the new branch forks from, an `ErrRemoteLeftover` otherwise.
  It never fetches.
- One ancestor helper both probes share: a commit missing locally (`cat-file -e <sha>^{commit}` fails) counts as not an ancestor; otherwise it returns `gitrepo.New(dir).IsAncestor(sha, ref)` unchanged, which already maps exit 1 to "no" and returns any other failure as an error.

In `add.go`, after `parentBranch` is resolved and before `createGitWorktree`, and only when neither `opts.SkipPush` nor `opts.SkipGit` is set, call `probeWarpLeftover` then `probeWeftLeftover` (passing `weftBranchAlreadyExists`), returning any error unchanged with an empty record.
Hold the `weftLeftover` answer for the step-12 replacement Card 5 adds; until then an archived leftover proceeds to today's push.
Step 11's warp push stays as it is: it is not gated by `SkipPush` or `SkipGit` today, and changing that is outside this plan's scope.
Extend the file header to say a leftover remote branch is resolved at pre-flight.

`add_remoteleftover_integration_test.go` (integration tag, hubs via `newFabricFixture`, empty `branch_prefix`), with a shared helper that pushes a commit onto an origin branch from a throwaway clone of the fixture's bare:
- unarchived weft leftover: add, plain Remove, push a commit onto origin's `<slug>-weft` from a clone → Add returns an `*ErrRemoteLeftover` naming the branch whose message carries `lyx fabric remove <slug>` and `git push origin --delete <slug>-weft`; the record is empty; no warp or weft worktree, no local warp or `<slug>-weft` branch, no portal and no launcher exists; origin's weft branch is unchanged.
- adopt path: after a plain Remove, recreate the local `<slug>-weft` at its old tip, push a commit onto origin's copy from a clone → Add refuses with an `*ErrRemoteLeftover` naming the branch, and the record is empty.
- warp leftover: add, commit and push warp work, squash-land it on the prime, plain Remove (the local warp branch is deleted) → Add refuses with an `*ErrRemoteLeftover` naming the warp branch, and no local warp branch exists afterwards.
- warp fast-forwardable: add, `Remove(l, slug, false, true)` (the remote weft branch goes, the remote warp branch stays), commit on the prime so its HEAD advances past origin's workless warp branch, re-Add → succeeds and origin's warp branch equals the new warp tip.
- `SkipPush`: the unarchived-weft-leftover setup, re-added with `AddOptions{SkipPush: true}` → succeeds locally, and origin's weft and warp branches are both unchanged; the warp one holds only because step 11's ungated push re-pushes the same prime HEAD the first Add pushed.

**ImpactSummary:** `Topology.Add` has three zero-`AddOptions` entry points: `fabriccli`'s `runAdd`, `battencli`'s create-worktree row, which passes the new refusal through `createRefusal` unchanged, and `hubforge.AddPair`, used by test suites in about ten packages. Each now makes two `ls-remote` reads before its first mutation and fails at pre-flight rather than at step 11 on an unreachable origin. No existing caller re-adds a removed slug, so the new refusal reaches only new behaviour; `SkipPush`/`SkipGit` callers are unaffected.
