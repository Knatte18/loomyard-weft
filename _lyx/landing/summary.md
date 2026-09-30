# loom start refuses a worktree with an unfinished merge

`lyx loom start` (and its root alias `lyx start`) used to spawn a driver onto a worktree whose merge had stopped on a conflict (a `UU` path, `MERGE_HEAD` present).
It checked no merge state before spawning or resuming a driver, and the recipe's Preflight row runs only at the head of the recipe inside the driver, so a resumed run never re-checked.

Now `start` refuses on its envelope with the new non-retryable kind `merge_in_progress`, carrying `conflicts` (the conflicted paths, an empty list when there are none), and exits non-zero.
The message names the remedy by case:

- A fabric merge is parked: resolve each path, run `lyx fabric merge-stage <path>...`, then `lyx fabric merge --continue` (or `--abort`), then re-run `start`.
- Merge state fabric did not start (a git merge, cherry-pick or squash): conclude or abort it with git.
  The fabric verbs refuse foreign state, so naming them there would send the operator into a second refusal.

## What changed

- `internal/fabricengine`: new read-only probe `MidMerge(l)`, vocabulary-neutral, with states none, parked (the fabric merge-state record exists) and foreign (`MERGE_HEAD` or unmerged entries on either side, no record).
  Conflicted paths come back in the one-repo form a fabric merge verb's `conflicts` uses, so `lyx fabric merge-stage` accepts them.
  The four git probes in `mergestate.go` are refactored into one shared reader that `foreignMergeStatePresent` now wraps.
- `internal/shedrun`: `StartMergeInProgressKind` declared beside `StartNotParkedKind`, so batten and other unattended callers tell the two by kind, not message text.
  Batten needs no change: it retries only `driver_not_parked`, so a `merge_in_progress` refusal halts instead of looping.
- `internal/loomcli`: the refusal in `runDriverSpawnAndWait`, with a `midMerge` seam on `loomCLI` for tests.
- Docs: `start`'s `Long` help, the loom entry in `docs/overview.md`, and the `l`-in entry-point list in `internal/fabricengine/doc.go`.

## Design decisions

- **The gate lives in `start`, not in `internal/preflight` or the Preflight row.**
  `start` is the verb that puts a driver to work.
  `step`, `run` and `shed step` stay unguarded because Landing's `mergeresolve` legitimately holds the pair mid-merge inside a step.
- **It refuses only when the invocation would spawn a driver or resume a parked one**, before the stale-marker removal, the resume line or any spawn.
  A live, working driver is left alone, because Landing's conflict-resolution session runs mid-merge and re-attaching to it must not be refused.
  A live strand with no park marker over a run halted at a hand-back keeps its `driver_not_parked` refusal.
- **A probe error refuses too.**
  A pair that cannot be opened or whose record location cannot be resolved is an undetermined answer, and spawning on it would reopen the bug.
- On every refusal the bootstrap lock is released and a stale park marker is left untouched.
- A fabric record wins over foreign-looking git state, since a parked fabric merge leaves `MERGE_HEAD` and unmerged entries of its own.

## Known consequences

- A record left by a crashed Landing `mergeresolve` session now blocks `start` until the operator runs `lyx fabric merge --abort`.
  The message cannot tell that origin from a hub merge-in.
- A merge-in started while a driver is actively stepping is not guarded.

## To check by hand

- Not run: an end-to-end `lyx loom start` over a real conflicted `lyx fabric merge-in`, and over a parked ly-drive driver with an attached tmux pane.
  Both are covered only by the test seams.
