# Add `lym fabric pull --hub` to pull the whole hub in one verb

Bringing a hub up to date took separate pulls of the prime's warp and weft and of Bolt, and some of them refused or raced while the orch, a dobby or a run was writing the board.
`lym fabric pull --hub`, run from any warp worktree of the hub, now pulls Bolt, the prime pair and every local beam pair, and reports per side what moved or why it stopped.
Plain `lym fabric pull` is unchanged, and `--hub` adds no new reach: it runs paths a plain pull and a board write already run, and the role guard that refuses `pull` from a task session refuses it with the flag too.

## Behavior

- Sides run in a fixed order: Bolt, the prime pair, then each local beam in `git worktree list` order.
  A side that stops never keeps the next side from running.
- Bolt is pulled through the new `Bolt.Pull` under the board write lock, then the existing pull-first path, which takes the push lock.
  A pull therefore never interleaves with a board write or a coalesced push.
- Each pair is pulled under the parent lock of its code branch, the lock a landing into that branch takes, so a pull never moves the branch under a landing in progress.
  The wait is unbounded except for Ctrl-C, the notice on stderr names the holder, and the lock is held for one pair's pull only.
- A local beam is a pair present on this machine whose origin record carries the beam mark, whatever machine owns it.
  Task pairs are not pulled, and nothing is pushed.
- A beam whose code branch has no upstream is reported `up_to_date`, since a beam made with no push has nothing to pull from.
- A beam whose origin record cannot be read is reported as a stopped beam side, and the remaining beams are still pulled.
- With `FABRIC_SKIP_GIT=1` every side is reported `up_to_date`.
- The envelope carries one entry per side under `sides`, with `kind` (`board`, `prime` or `beam`), `worktree`, `outcome` (`moved`, `up_to_date` or `stopped`) and `detail`.
  A stop's detail is the way forward its own path already names, such as the dirty-warp refusal for the prime.
  Any stop makes the envelope an error envelope that still carries every side, as `lym fabric reconcile` does, and the exit is non-zero.

## Design notes

- Git runs only in `internal/fabricengine` (`PullHub`); `internal/fabriccli` passes the parent-lock seam.
- `acquireBranchLock`, which `lym fabric anchor` used with a fixed run id, now takes the verb name.
  The lock owner's run id is `fabric-<verb>` and the wait notice names `lym fabric <verb>`, so the anchor verb's behavior is unchanged and the hub pull identifies itself as `fabric-pull`.
- This departs from PATTERN-parent-lock, which has a local parent branch move only in a run's Finalize.
  The plain `lym fabric pull` in the prime already moves that branch outside a run; the hub pull moves it the same way, but under the branch's parent lock.
- The prime's dirty warp stops the prime side with the existing refusal text; the hub pull stashes nothing.

Docs: the `pull` help text, and the `doc.go` of `internal/fabriccli` and `internal/fabricengine`.

## Check by hand

- The tests are hermetic fixtures and none ran a real hub: run `lym fabric pull --hub` once against a live hub while the orch is running, with origin ahead on Bolt, the prime and a beam.
- A long landing holds the parent lock for minutes and the hub pull waits for it; confirm the wait notice names the holder.
- The wait on a held parent lock is pinned by the CLI test `TestRunHubPull_WaitsForAHeldParentLock`; cancelling the command during the wait (Ctrl-C) leaves that pair stopped with the later sides still pulled, and no test covers it.
- A moving side after a stopped side is pinned in `TestPullHub_StoppedSideLeavesTheOthersPulled`; check that a pushed, origin-moved beam listed after a dirty prime is still reported `moved`.
