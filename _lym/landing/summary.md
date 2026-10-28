# Pull the whole hub with `lym fabric pull --hub`, safe beside a live orch

Getting a hub up to date used to take separate pulls of Bolt and of each pair's warp and weft, and some of those refused or raced while the orch, a dobby or a run wrote the board.
`lym fabric pull --hub` now pulls Bolt, the prime pair and every local beam pair in one call, from any warp worktree of the hub, and reports per side what moved or why it stopped.

## What changed

- `fabricengine.PullHub` pulls the sides in a fixed order: Bolt, then the prime pair, then each local beam pair in `git worktree list` order.
  A side that stops never keeps the next one from running.
  Each side is reported as `moved`, `up_to_date` or `stopped`, and a stop's detail is the way forward its own path already names.
- `Bolt.Pull` takes the board write lock and runs the pull-first path a board write runs, which also takes the board's push lock.
  A Bolt pull therefore never interleaves with a board write or a coalesced push.
- Each pair goes through the existing `Fabric.Pull` while the caller's `PairLock` holds the parent lock of its code branch, the lock a landing into that branch takes.
  The lock is held for that one pair only.
- A local beam is a pair present on this machine whose origin record carries the beam mark, whatever machine owns it.
  A beam whose code branch has no upstream is reported `up_to_date` without taking the lock.
  Task pairs are left to their own runs.
- In the CLI, `--hub` on `lym fabric pull` resolves the prime and passes the parent-lock seam `lym fabric anchor` already used.
  `acquireBranchLock` now takes the verb, so the lock owner's run id (`fabric-pull`) and the stderr wait notice name the right verb.
  The envelope's `sides` lists `kind`, `worktree`, `outcome` and `detail` per side.
  Any stopped side makes the output an error envelope that names the stopped sides and still lists every side.
- The `lym fabric pull` help gained the `--hub` paragraph, and its existing bullets were tightened.
  `internal/fabriccli/doc.go` and `internal/fabricengine/doc.go` each gained a hub-pull section.

## Design decisions

- `--hub` is a flag on the existing pull verb rather than a new verb, so the operator keeps one pull verb.
  The role guard that refuses `pull` from a task session refuses it with the flag too.
- A held parent lock is waited for, as `lym fabric anchor` waits, instead of being reported as a stop.
  Ctrl-C ends the wait through the command context.
- The board write lock is held for the Bolt side only, never across the pair pulls, so a long pair pull never stalls board writes.
- A pair whose records side could not reach origin is reported `stopped`, since `Fabric.Pull` only warns about it and would otherwise read as up to date.
- Moving the prime's or a beam's branch outside a run's Finalize is a departure from `PATTERN-parent-lock`, recorded as a `Pattern-Departure` trailer on the engine commit.
  The plain pull in the prime already moves that branch outside any run.
  The hub pull does it under the branch's parent lock, so it never moves a branch while a landing is in progress.

## Tests

`internal/fabricengine/hubpull_integration_test.go` (tier 1) pins:
- every moved side pulled, then all sides `up_to_date` with no mutations;
- a dirty prime stopping with `ErrWarpDirty` while Bolt still moves;
- a beam with no upstream taking no lock;
- each pair's pull falling between taking and releasing its lock;
- `Bolt.Pull` waiting for a held board write lock.

`internal/fabriccli/hubpull_integration_test.go` (tier 1) drives the verb from a task pair through the real parent lock.
It checks the clean and the stopped envelope, the exit code, and that no parent lock is left held.
