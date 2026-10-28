# Add `lym fabric pull --hub` to pull the whole hub from origin

Bringing a hub up to date took separate pulls of the prime's warp and weft, of Bolt and of each beam, and some refused or raced while the orch, a dobby or a run wrote the board.
`lym fabric pull --hub` replaces them with one verb, run from any warp worktree of the hub.
It reports each side under `sides` as `board`, `prime` or `beam`, with the worktree, an outcome (`moved`, `up_to_date` or `stopped`) and a detail.
A stopped side's detail names its way forward.
Any stopped side makes the envelope an error envelope that still carries every side, and the verb exits non-zero.
`lym fabric pull` without the flag behaves as before.

## Design

- Sides run in a fixed order: Bolt, the prime pair, then each local beam in `git worktree list` order.
  A side that stops never keeps the next one from running.
- Bolt is pulled through a new `Bolt.Pull`, which takes the board write lock and then runs the existing pull-first path, which holds the push lock.
  The pull therefore never interleaves with a board write or a coalesced push.
- Each pair is pulled through `Fabric.Pull` while holding the parent lock of its code branch, the lock a landing into that branch takes.
  The lock seam is the one `lym fabric anchor` uses: it waits with no time limit, prints the holder while waiting, ends on Ctrl-C, and is held for one pair only, never across sides.
- A local beam is a pair present on this machine whose origin record carries the beam mark, whatever machine owns it.
  Task pairs and pairs of other machines are left to their own runs.
- A beam whose code branch has no upstream is reported `up_to_date`, since otherwise a beam made with no push would fail every hub pull.
- A dirty prime warp stops the prime side with the existing `ErrWarpDirty` text; the hub pull stashes nothing.
- Git stays in `internal/fabricengine` (`PullHub`); `internal/fabriccli` only supplies the lock seam and renders the report.
- The role guard that refuses `pull` from a task session refuses it with the flag too.

## Pattern departure

`PATTERN-parent-lock` says a local parent branch moves only in a run's Finalize.
The hub pull moves the prime's and a beam's branch outside a run, as the plain pull in the prime already does, but under that branch's parent lock, so it never moves a branch under a landing in progress.
The commit carries a `Pattern-Departure: PATTERN-parent-lock` trailer.

## Docs

The fabric CLI and engine `doc.go` files describe the verb and `PullHub`.
`docs/overview.md` is unchanged, as no module row changed.

## To check by hand

New tests are hermetic hubforge integration tests covering moved, level, stopped, no-upstream and parent-lock-wait cases.
A pull against a real hub with a live orch, and a long landing holding the parent lock, were not run; the pull waits for such a landing and names the holder.
