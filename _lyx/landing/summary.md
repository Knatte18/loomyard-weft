# lyx temp root, test temp cleanup and a `lyx cleanup` verb

lyx and its tests no longer leave temp garbage in the system temp directory, and one operator verb tidies the repo's other leftover state.
On a tmpfs `/tmp`, leaked gitconfigs, fixture templates and prebuilt `lyx` binaries had grown to 8.5 GB and broke hibernation.

## Temp root

- New leaf package `internal/lyxtemp` owns a per-user root in the system temp directory.
  The root must be a real directory; off Windows it must also be mode 0700 and owned by the current user.
  Otherwise the caller fails with an error naming the path, and never falls back to the bare system temp directory.
- Each process gets one run directory under the root and holds an `internal/lock` lock on a file inside it for its life.
  The owner unlocks and removes the directory on normal exit.
- Creating a run directory first sweeps the root: a prefixed directory older than a 10-minute grace is removed when it has no lock file or its lock can be taken.
  A live owner's directory is never touched, whatever its age.
- A child that inherits a run directory as `TMPDIR` creates its own beside it, so socket paths stay short and a child never sweeps inside its parent.
- `tmuxkit.Main` uses the process run directory as its private directory and removes it after `m.Run` on every platform.
  `gitkit` (gitconfig and repo template), `hubforge`, `lyxbin`, `internal/gatecli`'s prebuild and `cmd/testtiming` create their temp directories there.
- Every test package now has a `TestMain` calling `tmuxkit.Main`, and the tmux-isolation scan in `cmd/lyx` requires it.
- A new scan in `cmd/lyx/baretemp_test.go` fails a bare `os.MkdirTemp`/`os.CreateTemp` with an empty or `os.TempDir()` dir outside `lyxtemp`; an allowlist entry names one file, one call and a reason.
- `testtiming` now attributes leftovers by the inherited `LYX_TESTTIMING_PACKAGE` marker instead of a per-package `TMPDIR`.

A panicked or timed-out test binary skips the rest of `TestMain`, so its run directory is removed by the next start's sweep, not at exit.
Leftovers that older code created directly in the system temp directory are not swept; remove them once by hand.

## `lyx cleanup`

`lyx cleanup` runs from the prime worktree only and is a dry run unless given `--apply`.
It calls each owning module's function in-process, in this order, and a failing segment does not stop the rest; the exit is non-zero if any failed:

1. `lyx batten cleanup`: for a slug whose status is `done` and whose run lock can be taken, removes the records run directory in one pushed records commit and empties the slug's `.lyx` scratch except its lock files.
   It skips every other state, a held run lock and an undecodable status, and reports each skip.
   It also empties an orphan scratch directory older than one day.
   A run directory deleted on disk but still tracked counts only when the committed status is `done`.
2. `lyx board prune`, unchanged.
3. `lyx orch cleanup`: removes every handoff except the one `State.LastHandoff` names.
   It holds the orch state lock and does nothing unless the cycle phase is idle.
4. `.scratch/`, owned by the new `internal/cleanupcli`: removes a direct child whose newest inner modification is older than 14 days and which no open board entry cites.
   A board read failure fails the segment and removes nothing.
5. Logger's retention sweep, with the bounds `cmd/lyx`'s exit sweep uses.
6. The temp root's sweep.

The verb adds no `--force` and removes nothing a segment's own verb would refuse to.
Batten's done-slug refusal now names `lyx batten cleanup --apply` for the run directory.
`fsx.NewestModTime` is new; logger's sweep gained a dry run, and `loggerconfig` and `shedrun` gained small helpers the segments use.

## Decisions to know

- `lyx fabric prune` and `lyx fabric cleanup` are deliberately not segments: their debris has the shape of a pair that batten's teardown has half removed and will finish on resume, and neither shares a lock with that teardown.
- The 10-minute grace, the one-day orphan age and the 14-day `.scratch` age are chosen values, not operator-given.
- "Old orch notes" is read as the handoff notes.
- `PATTERN-lyx-temp-root` is new, `PATTERN-leaf-packages` and `PATTERN-test-isolation` are updated, and `docs/overview.md`, `internal/orchengine/doc.go` and the `doc.go` of each new or touched package are updated.

## To check by hand

- `TestSweep/two_concurrent_sweeps_both_succeed` in `internal/lyxtemp` failed once in the integration suite and passed on rerun; check it for a real race.
- `lyx cleanup` run from outside the prime is not covered by a test; check that it refuses and names the prime.
- The `testtiming` re-sign path misses an arrow in its flow; check `cmd/testtiming` for the missing step.
- A tracked run directory with no committed `status.json` is a seed case in batten cleanup that has no test; check that it is skipped and reported, not committed.
- A dry run of `lyx orch cleanup` created the orch state lock file when it was absent; the fix is in `internal/orchengine`, so check that a dry run on a fresh orch directory creates nothing.
- `reedengine`'s socket-file tests keep short `os.MkdirTemp` paths for socket length; check that they are allowlisted or moved, and that `reed-socket-test-fd-sweep` does not conflict.
- Worktrees branched before this lands keep leaking the old way until they rebase.
- Not run: `lyx cleanup --apply` against a real hub and a real records remote; run it once from the prime.
