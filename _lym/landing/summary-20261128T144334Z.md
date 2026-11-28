# clone --reset refuses a hub holding unpushed records or ignored work

`lym fabric clone --reset` used to remove a hub's unpushed records and board commits, uncommitted Bolt changes and ignored files with the hub.
Losing that work is not acceptable.
`--reset` now refuses when it would lose any of it and names what would be lost.
A new flag, `--discard-work`, accepts the loss.

## What changes

- **More stores are read.**
  Before removing a hub, `--reset` now checks the prime's records store and the `_bolt` checkout as well as the prime code store.
  In each of the three, uncommitted or untracked changes and ignored files count as work.
  In the code store and the records store, a stash and commits origin lacks count too.
  The board's commits are counted in the records store, since `_bolt` is one of its worktrees.
- **Ignored files.**
  A refusal names up to five ignored paths per store and counts the rest.
  A wholly ignored directory is one entry.
  Links are skipped, since their content lives at the target, and so are the `.lym` cache directories and the `.weft` records lock directories.
- **`--discard-work`.**
  It goes with `--reset` and lets it remove a hub whose stores hold work.
  Passed without `--reset`, it is refused before anything is read.
  A pair worktree or an unfinished teardown still refuses, flag or not: those mean the hub is in use.
- **Ways forward.**
  A refusal over store work alone offers four: push or commit the work, remove it, move the hub aside with the `mv` and `worktree repair` command,
  or run the `--reset --discard-work` clone, which the refusal spells out.
  A refusal that names a pair or a teardown keeps the old way forward: land or remove the work, or move the hub aside.
  A plain clone into an unfinished hub that holds work uses the same wording.
- **Seed refusal.**
  When a board push is refused because origin holds no board, the way forward is now the `--single-repo` clone with `--reset --discard-work`.
  The hub's records store holds the unpushed seed, which plain `--reset` now refuses.
- Package docs of `fabricengine` and `fabriccli`, the `clone` help text and scenario F12 of `tools/sandbox/SANDBOX-FABRIC-SUITE.md` are updated.

## Design decisions

- **The records store is now read.**
  The earlier design left it unread so that `--reset` could replace a hub whose seed stopped before its push.
  That replacement now goes through `--discard-work`, which the seed refusal and the reset refusal both name.
  A clone killed before its push leaves its own commits unpushed, so replacing that hub takes the flag too.
  A finished clone commits its per-worktree module configs on the records branch with no push.
  Until a later push carries them, a plain `--reset` of that hub is refused and names those commits.
- **Every ignored path counts as work.**
  Git cannot tell a build output from a kept secret such as `.env`.
  So a hub with build outputs refuses plain `--reset`.
  The refusal names those paths, and `--discard-work` is how to proceed.
- **The flag does not cover a hub in use.**
  Removing a hub that holds a pair's worktree or an unfinished teardown pulls the ground from under a run or loses archive tags only this machine has.
  Those still have one way forward: landing the work or moving the hub aside.
- `runCloneWithReset` takes its flag values as one `cloneFlags` struct instead of a growing list of positional parameters.

## To check by hand

- **Test fixtures and pushes (`clone_singlerepo_integration_test.go`).**
  The new `pushRecordsStore` helper runs `git push origin` in a one-repository hub's records store through its push refspecs.
  Check that it pushes the board and records branches fast-forward on a fresh `hubforge` hub,
  so `ResetProceedsOverACommitOriginHoldsUnderATagAlone` and the records and board rows start from a hub with nothing unpushed.
- **Existing refusal tests now pass `DiscardWork`.**
  `TestCloneHub_ResetRefusalKeepsExistingHub`, `TestCloneHub_ResetSubpathCheckReadsRecordedAnchor` and two subpath subtests of the one-repository test pass `DiscardWork: true`.
  A bare `CloneHub` leaves uncommitted board files and unpushed records, which plain `--reset` now refuses before the check each test is about.
  Check that each still reaches its intended refusal.
  For the same reason, the `RealHub` cells of `TestCloneHubReset` pass `DiscardWork`.
  `TestCommand_HubWithoutItsBoltNamesTheResetClone/TwoRepository` and `TestCloneHub_ShortnameRecordLifecycle` push the records store first.
  No assertion of these tests changed.
- **Unfinished hub (`TestCloneHub_HubExistsCheckPrecedesProbeInTwoArgForm`).**
  The test now asserts both cases: the clone's leftover work named with the `--discard-work` clone,
  and then, after a commit and `push --all`, the plain `--reset` clone.
- **Ignored-path filter (`ignoredWork` in `internal/fabricengine/clone.go`).**
  Check that no other directory lym creates in a prime, records store or Bolt checkout shows up as ignored work on an ordinary hub.
- **Sandbox scenario F12.**
  Run the updated `--reset` and `--discard-work` steps.

## Merge danger

**Door:** two-way
**Blast radius:** fabric
