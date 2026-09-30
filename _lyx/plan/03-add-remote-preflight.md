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
- One ancestor helper both probes share: a commit missing locally (`cat-file -e <sha>^{commit}` fails) counts as not an ancestor; otherwise `merge-base --is-ancestor`, exit 1 meaning no, and any other failure returned as an error.

In `add.go`, after `parentBranch` is resolved and before `createGitWorktree`, and only when neither `opts.SkipPush` nor `opts.SkipGit` is set, call `probeWarpLeftover` then `probeWeftLeftover` (passing `weftBranchAlreadyExists`), returning any error unchanged with an empty record.
Hold the `weftLeftover` answer for the step-12 replacement the next card adds; until then an archived leftover proceeds to today's push.
Extend the file header to say a leftover remote branch is resolved at pre-flight.

`add_remoteleftover_integration_test.go` (integration tag, hubs via `newFabricFixture`, empty `branch_prefix`), with a shared helper that pushes a commit onto an origin branch from a throwaway clone of the fixture's bare:
- unarchived weft leftover: add, plain Remove, push a commit onto origin's `<slug>-weft` from a clone → Add returns an `*ErrRemoteLeftover` naming the branch whose message carries `lyx fabric remove <slug>` and `git push origin --delete <slug>-weft`; the record is empty; no warp or weft worktree, warp branch, portal or launcher exists; origin's weft branch is unchanged.
- adopt path: after a plain Remove, recreate the local `<slug>-weft` at its old tip, push a commit onto origin's copy from a clone → Add refuses with an `*ErrRemoteLeftover` naming the branch, and the record is empty.
- warp leftover: add, commit and push warp work, squash-land it on the prime, plain Remove (the local warp branch is deleted) → Add refuses with an `*ErrRemoteLeftover` naming the warp branch, and no local warp branch exists afterwards.
- `SkipPush`: the unarchived-weft-leftover setup, re-added with `AddOptions{SkipPush: true}` → succeeds locally and origin's weft branch is unchanged.

**ImpactSummary:** `Topology.Add` gains two read-only pre-flight probes, skipped under `SkipPush`/`SkipGit`; callers see a new typed refusal and no other change.
