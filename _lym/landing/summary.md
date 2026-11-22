# Prune deleted remote branches on fetch so a cancelled slug can run again

A cancelled run's slug could not start again.
`lym batten cancel --remove` deletes the records branch on origin, but the records repository kept the remote-tracking ref of that branch.
`fabricengine.OriginCodeTip` read the stale ref, found the old run's last transition and judged its code tip missing.
Dobby then refused the start with "the code tip … is not on origin yet", though the tip was on origin.
The refusal also pointed at a code push that would never land.

## What changes

- `gitrepo.Repo.Fetch` now prunes the remote-tracking ref of every branch origin deleted, so every reader of the fetched refs stops seeing a deleted branch as present.
  After the plain fetch it runs `git fetch --prune --no-tags origin <refspecs>` under the same remote-refs hold.
  The refspecs are origin's configured ones whose destination is under `refs/remotes/`.
  The prune is limited to them so it never deletes a local tag, which a bare `--prune` would do when a configured refspec fetches into `refs/tags/`.
  A checkout with no `origin` is fetched and not pruned.
  A failed fetch error names the command that failed, with each refspec single-quoted.
- `battencli` judges a code-tip wait again after a fetch.
  When the act rule's verdict is the code-tip wait, `hubRunVerdict` fetches the run's records and code refs from origin, including the prime's code store, and judges a fresh read.
  A failed fetch keeps the first verdict and its refusal.
  A cancelled slug is therefore released by the next verb that asks, with no manual `git fetch --prune`.
- The code-tip refusal names the real cause when no pair of the run stands on this machine.
  `runfsm.Inputs` and `runfsm.Verdict` gain `CodeTipPairAbsent`, which `pairteardown` sets from whether a pair stands here.
  In that case the refusal says the transition may be read from a records branch origin has deleted, and the way forward is `lym fabric pull` in the prime before the code-push advice.
  A refusal with a pair present reads as before.
- `cmd/lym/checkedcall_test.go` pins the two marked raw git calls `Fetch` now has.

## Tests

- `internal/gitrepo/fetch_integration_test.go`: a records-store fetch prunes the tracking refs of deleted branches and keeps local tags.
- `internal/battencli/arm_runverdict_test.go`: rows for the fetch before the second judgement, a failed fetch, and a read that fails after the fetch.
- `internal/runfsm/act_test.go`: rows for the pair-absent refusal, with and without a tip and machine.
- `internal/pairteardown/runread_test.go`: `CodeTipPairAbsent` is set from the pair's presence.

## Design decisions

- Pruning in `Fetch` rather than only in `lym batten cancel --remove` covers every path that leaves a stale ref, including a branch deleted from another machine.
- The prune names refspecs instead of passing a bare `--prune`: a bare prune also removes local tags that no configured refspec fetches into `refs/remotes/`.
- Only `origin` is read for the prune; every store in the repository uses it.
- The fetch in `hubRunVerdict` also fetches the code store when a pair stands, which repeats the code-side fetch on that rare path; a guard would move pair knowledge into the closure.

## To check by hand

- The prune of `Fetch` must never delete a local tag: confirm on a checkout whose remote has a refspec fetching into `refs/tags/` (`internal/gitrepo/pull.go`).
- The merge-danger call below is wider than a battencli-only change because `Fetch` is shared by every caller of `gitrepo`: confirm no caller relies on a tracking ref of a branch origin deleted.
- Not run: the reproduction from the report, a cancelled `harden-fabricengine`-style slug released again against a real origin.

## Merge danger

**Door:** one-way
**Blast radius:** gitrepo
