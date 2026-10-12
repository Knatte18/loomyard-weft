# lyx temp root, run-directory sweep and `lyx cleanup`

lyx and its tests leaked GBs into the system temp directory: a built `lyx` binary per smoke run, a gitconfig file per test binary, and fixture templates that were never removed.
On a tmpfs `/tmp` this grew to 8.5 GB and made hibernation fail.
This change gives lyx one temp root with locked, sweepable run directories, and adds `lyx cleanup` for the repo's other routine leftovers.

## Temp root (`internal/lyxtemp`)

- A new leaf package owns a per-user root in the system temp directory.
  The root must be a real directory; off Windows it must also be mode 0700 and owned by the current user.
  Otherwise the package returns an error naming the path, and callers fail rather than fall back to the bare system temp directory.
- Each process gets one run directory under the root, with an exclusive `internal/lock` lock held for the process's life.
  The owner unlocks and removes it on normal exit.
- Creating a run directory first sweeps the root.
  The sweep removes only prefixed, non-link directories older than a 10-minute grace whose lock file is absent or can be taken.
  It never reads inside a live run directory.
- A child that inherits a `TMPDIR` that is a run directory creates its own beside it, not inside it.
  This detection moved here from `tmuxkit`.
- `tmuxkit.Main` uses the process run directory as its private directory and removes it after `m.Run` on every platform.
  `gitkit`, `hubforge`, `lyxbin`, `ide`, the reed socket tests, `internal/gatecli`'s prebuilt `lyx` and `cmd/testtiming` create their temp files there.
- Every test package now has a `TestMain` calling `tmuxkit.Main`.
  `cmd/lyx`'s tmux-isolation scan requires it and is renamed `TestTmuxIsolation_EveryTestPackageRunsThroughTmuxkitMain`.
- A new scan, `cmd/lyx/baretemp_test.go`, fails `os.MkdirTemp`/`os.CreateTemp` with an empty or `os.TempDir()` dir outside the temp package.
  Admitted sites sit in an allowlist in the test, each with a reason.
- `cmd/testtiming` attributes leftovers by the inherited `LYX_TESTTIMING_PACKAGE` marker instead of a per-package `TMPDIR`.

## `lyx cleanup`

A new top-level verb, prime worktree only, dry run by default and `--apply` to remove.
It calls each owning module's function in-process, in order, and a failing segment does not stop the later ones; the exit is non-zero if any failed.

1. `lyx batten cleanup`: for each slug whose status is `done` and whose run lock can be taken, removes the records run directory in one pushed records commit, then empties the slug's `.lyx` run scratch except lock files.
   It also empties aged (over one day) orphan scratch directories.
   Other states, a held lock and an undecodable status are skipped and reported.
   The done-slug refusal in `lyx batten run` now names `lyx batten cleanup --apply` (the `batten-reused-slug` friction).
2. `lyx board prune`.
3. `lyx orch cleanup`: removes every handoff except the one `State.LastHandoff` names, holding the state lock and only in the idle phase.
   `internal/orchengine/doc.go` is updated to match.
4. `.scratch/`, owned by the new `internal/cleanupcli`: removes direct children untouched for 14 days that no not-done board entry cites.
5. Logger retention, with the bounds the exit sweep picks.
6. The temp root's sweep.

Supporting changes: `fsx.NewestModTime`, a dry run on logger's sweep, `loggerconfig` exposing the exit sweep's bound choice, and `shedrun` accessors for the run-records roots.

## Design decisions to know

- `lyx fabric prune` and `lyx fabric cleanup` are not part of `lyx cleanup`.
  Their debris has the shape of a pair that batten's teardown has half removed, and they share no lock with it, so scheduling them could pre-empt the teardown.
- The 10-minute grace, one-day orphan age and 14-day `.scratch` age are chosen values, not given ones.
- A panicked or timed-out test binary skips `TestMain`'s remaining code, so its run directory is removed by the next start's sweep, not at exit.
- Leftovers created directly in the system temp directory by older code are not swept; remove them once by hand.
- Docs: new `pattern/PATTERN-lyx-temp-root.md`, updates to `PATTERN-leaf-packages.md` and `PATTERN-test-isolation.md`, `doc.go` for the touched packages, `docs/overview.md`, and the sandbox core suite.

## To check by hand

- `internal/lyxtemp` `TestSweep/two_concurrent_sweeps_both_succeed` failed once in the integration suite and passed on rerun; it is flaky, so check the concurrent-sweep path for a race.
- `lyx cleanup` run outside the prime worktree should refuse naming the prime; the refusal has no test (`cmd/lyx` cleanup verb).
- `testtiming`'s resign arrow is missing from its documentation and should be checked against `cmd/testtiming`.
- `lyx batten cleanup` for a tracked-but-deleted run directory with no `status.json` in the records HEAD tree: confirm it is skipped and reported, not seeded or committed.
- The gate lines for `./internal/logger_test` and `./internal/loggerconfig_test` in the plan named non-directories and were reported `[setup failed]`; the real packages passed.
- Run `lyx cleanup` (dry run, then `--apply`) from the prime to see the segments on real state; it was not run against live data.
- `reedengine`'s socket-file tests use short temp paths for socket length and may need an allowlist entry in the bare-temp scan.
- Worktrees branched before this lands keep leaking the old way until they rebase.
