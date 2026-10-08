# Git robustness: stale pack index, background gc, network timeouts, reused slugs

`git-robustness` · Task · bug, enhancement, fabric, batten, stencil · running · Plan-Bouncer

## Goal

- `object not found` (#468): the reindex-and-retry of `lookupObjectRetrying` covers every object-resolving go-git call in one place, `Status()` in `internal/gitrepo/worktree.go` (`UntrackedFiles`, `WorktreeChangedFiles`), tree walks and diffs included, with a regression test: build the pack index, commit with the CLI, `git repack -d` and `git prune-packed`, then call `UntrackedFiles`.
- The hub's shared object store stays below the auto-gc threshold, or auto-gc runs at known points, so no background repack lands under a live step.
- Every git network call (push, fetch) has a timeout, so a batten run never hangs after a laptop sleep.
- `batten run` on a slug whose earlier run is done refuses by name instead of reporting ok and doing nothing.
- `gitrepo.HeadContains` answers ancestry with a merge-base or a bounded walk instead of walking the whole history on every source-ahead seed pass.

The merged notes keep their own Seen sections below.

## From `merge-in-object-not-found`

Imported from [issue #468](https://github.com/Knatte18/loomyard/issues/468).

## Seen

Run `binary-change-and-rules`, Publish, 2026-10-08T18:41Z (trace `trace-20261008T184138Z-82c70335c18c4f83-634197.log` in the task worktree).
After `fabric: mutation kind=merge_staged` and a conflict agent finishing `done`, Publish failed with:

`landingshed: Publish: merge-in against parent branch "main": mergeresolve: list untracked files (attempt 1): fabricengine: list untracked files: gitrepo: worktree status: object not found`

The worktree was left mid-merge with `internal/battencli/wire.go` unmerged (UU) and every merged path staged.
The driver's repair re-step then stopped `blocked` on "the task worktree has uncommitted changes before the merge-in", naming the staged merge itself.
`git fsck --connectivity-only` on the pair was clean, so the object exists for git; the failure is in lyx's own worktree status read during a merge.
The machine had been hard-reset about 30 minutes earlier; that may or may not matter.

## Workaround

The orch ran `lyx fabric merge --abort` in the task worktree and `lyx loom resume`, so Publish redoes the merge-in from a clean tree.

## Expected

The untracked-file listing works mid-merge, or a failed merge-in aborts its own staged merge before refusing, so the run never halts on its own leftover merge, and the dirty-tree refusal names `lyx fabric merge --abort` when a merge is in progress.

## Root cause (orch investigation, 2026-10-08)

lyx's cached go-git handle read a stale pack index after an automatic `git gc` repacked the shared object store mid-step.
The new pack `pack-4160613d…` landed at 20:42:06 CEST while the conflict agent ran, and HEAD `64df647cf` and its tree then existed only in that pack, so the handle's index missed them and their loose copies were gone.
`internal/gitrepo/doc.go` already documents this failure, and `lookupObjectRetrying` handles it with a pack fingerprint check, `storer.Reindex()` and one retry, but the go-git `Status()` calls in `internal/gitrepo/worktree.go` (`UntrackedFiles`, `WorktreeChangedFiles`) bypass that wrapper.
The hard reset was not the cause: the pack was written 30 minutes after it and `git fsck` is clean.

## Fix

Apply the reindex-and-retry in one place that every object-resolving go-git call goes through, `Status()`, tree walks and diffs included, and add a regression test: build the index, commit with the CLI, `git repack -d` and `git prune-packed`, then call `UntrackedFiles`.

## From `shared-store-auto-gc`

## Seen

The hub's shared object store (`loomyard/.git`, used by the prime and every pair) holds about 8,450 loose objects, above git's default `gc.auto` threshold of 6,700.
They survive each repack, most likely because they are unreferenced and kept until the two-week prune expiry, so nearly every commit, merge or fetch in any worktree starts a background auto-gc.
On 2026-10-08 new packs appeared at 16:40, 18:07, 19:34, 19:52, 20:00 and 20:42, and the 20:42 one broke a Publish merge-in (#468).

## Idea

lyx keeps the shared store below the auto-gc threshold, for example by pruning unreferenced loose objects at landing, or sets `gc.auto` for the hub's store so repacks run at known points rather than under a live step.

## From `git-network-timeout`

Imported from [issue #458](https://github.com/Knatte18/loomyard/issues/458).

## What happened

A `lyx batten run review-lens-fan` began a best-effort status push to the weft remote at 14:11:21; the laptop hibernated four seconds later.
After resume the push hung on a dead HTTPS connection (`git send-pack` in `futex_do_wait`) for 2.5 hours, and the batten run sat at Run-Shed without starting the child loom, with no notice to the parent.
Killing the `git push` let batten log `status push failed, next transition will catch up` and go on.

## Expected

Git network calls from lyx (push and fetch, best-effort or not) run under a timeout, or with `http.lowSpeedLimit`/`http.lowSpeedTime` set, so a dead connection fails within minutes and the best-effort path takes over.


## From `batten-reused-slug`

Imported from [issue #447](https://github.com/Knatte18/loomyard/issues/447).

Seen: `lyx batten run webster-recovery --window` returned `{"existing":false,"ok":true,...}` and opened a window, but no pair was created and no loom run started.
`_lyx/shed/webster-recovery/status.json` held a batten run from 2026-10-02 for an earlier task of the same slug, in state `done` at Worktree-Teardown, so the new run read that status as finished.
The operator noticed only because no VS Code window opened.

Expected (operator, 2026-10-08): a slug whose task is done and pruned from the board is free, and a new task with the same slug runs as a fresh run.
A finished run's status must not stand in the way: it is archived or dropped once its task is done, so `batten run` for the new task starts at Worktree-Create.

## Proposal

- `batten run` on a slug whose status is terminal and done treats the slug as free: it archives or drops the old `_lyx/shed/<slug>/` and starts at Worktree-Create.
  A halted or failed status stays, since it is meant to be resumed.
- Removing finished runs' status directories is batten's own cleanup verb, which the repo-wide cleanup command in note `lyx-cleanup` calls.
  The board never touches shed state: `lyx board prune` stays board-only.
- A running batten run's directory is never removed, so the cleanup skips any slug whose status is not terminal.

Workaround: the task was renamed to `webster-dead-ends`.

## From `headcontains-full-walk`

## Seen

The PR-Gate review of `binary-change-and-rules` (PR #469) found that `gitrepo.HeadContains` (`internal/gitrepo/blobread.go`) walks HEAD's whole history through go-git whenever a seed pass finds a source-ahead stencil.
In a long-history repository that cost lands on every command until the pair syncs with main.

## Idea

Answer the ancestry question with a merge-base or a walk bounded by commit date or depth instead of a full history walk.
