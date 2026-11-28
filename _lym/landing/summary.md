# clone --reset refuses a hub holding unpushed records or ignored work

`lym fabric clone --reset` removed a hub's unpushed records and board commits, uncommitted Bolt changes and ignored files along with the hub.
The cause: `hubWorkOriginLacks` read only the prime code store.
`--reset` now refuses when it would lose that work, and names it.
A new flag, `--discard-work`, accepts the loss.

## What changes

- **More stores are read.**
  `--reset` now checks the prime code store, the prime's records store and the `_bolt` checkout.
  In each, uncommitted or untracked changes and ignored files count as work.
  In the code store and the records store, a stash and commits origin lacks count too.
  The board's commits are counted in the records store, since `_bolt` is one of its worktrees;
  commits only `_bolt`'s own HEAD holds, as on a detached HEAD, are counted from `_bolt`.
- **Ignored files.**
  Every ignored path counts as work except links and the files lym writes itself.
  A refusal names up to five paths per store and counts the rest.
- **`--discard-work`.**
  With `--reset`, it removes a hub whose stores hold work.
  Without `--reset`, it is refused before anything is created.
  A pair worktree or an unfinished teardown still refuses, flag or not.
- **Ways forward.**
  A refusal over store work alone offers: push or commit the work, remove it, move the hub aside,
  or run the `--reset --discard-work` clone, which the refusal spells out.
  A refusal that names a pair or a teardown offers landing or removing that work, or moving the hub aside.
  A plain clone into an unfinished hub that holds work gives the same ways forward.
- **A store that cannot be read** refuses `--reset`, with or without `--discard-work`, naming the move aside.
- **Hub without its Bolt checkout.**
  When its stores hold work and nothing uses the hub, the replacing clone named by the refusal takes `--discard-work`,
  and the refusal names the work that clone loses.
  For a hub in use, the refusal names what uses it.
- **Seed refusal.**
  When origin holds no board, the way forward is now the `--single-repo` clone with `--reset --discard-work`,
  since the records store holds the unpushed seed.
- Package docs, the `clone` help text and both sandbox suites are updated.

## Design decisions

- **A finished clone is not reset without the flag until its records are pushed.**
  It commits its per-worktree module configs on the records branch with no push.
  A clone stopped before its push leaves commits there too.
- **Every ignored path counts, because git cannot tell a build output from a kept secret.**
  A hub with build outputs therefore refuses a plain `--reset`.
- **What counts as written by lym:** the `.lym` and `.weft` directories;
  in the prime, `.vscode/settings.json`, `.vscode/tasks.json` and the warp exclude list's entries, all under the anchor;
  in `_bolt`, `.board-rendered.json`, `board.lock` and `board.push.lock`.
  When the prime's anchor or the fabric config cannot be read, fewer paths are skipped, so more is named.

## To check by hand

- **Skipped files can hold your own content.**
  `internal/fabricengine/clone.go`, `lymWrittenInPrime`: a skipped file is skipped whole,
  so your own keys in `.vscode/settings.json` or `.claude/settings.local.json`,
  and anything you added to the warp exclude list, are removed unnamed.
  Decide whether that is acceptable.
- **Older hubs may still be refused over empty lock files.**
  `.gitrepo-push.lock` or `board.json.swaplock` left in `_bolt` by an earlier layout are named as work.
- **Untested branch.**
  No test covers `RefuseHubWithoutBolt` on a hub in use.
- **Root cause.**
  `internal/fabricengine/clone.go`, `hubWorkOriginLacks`: check that the cause stated at the top is right.
- **`--discard-work` pass-through.**
  `internal/fabriccli/fabric.go`: the `CloneDiscardWorkWithoutReset` row drives the flag without `--reset`.
  Check that this is enough, as no CLI test passes both flags.
- **Commits only `_bolt`'s HEAD holds.**
  `internal/fabricengine/clone.go`, `storeWorkOriginLacks` counts them;
  check the count and its `CommitOnADetachedBoltHead` row.
- **Unreadable store.**
  `internal/fabricengine/clone.go`, `checkResetTarget`: check that its refusal names the move aside.
- **Core sandbox suite.**
  `tools/sandbox/SANDBOX-CORE-SUITE.md`, step S6 says when a repeated clone takes `--reset --discard-work`.
  Check that wording.
- **Sandbox scenarios.**
  Run the updated `--reset` and `--discard-work` steps of F12 in `tools/sandbox/SANDBOX-FABRIC-SUITE.md`, and S6 of the core suite.

## Merge danger

**Door:** two-way
**Blast radius:** fabric
