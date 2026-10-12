# Decision record — test-tmp-cleanup

## Goal

lyx and its tests leave no temp garbage in the system temp directory, and one operator verb, `lyx cleanup`, runs the routine cleanups of the repo's leftover state named in Decisions D7, with a dry run by default.

## Scope

In:

- A lyx-owned per-user temp root in the system temp directory, with one locked run directory per process and a lock-guarded sweep of dead owners' run directories.
- Moving every lyx temp creation into that root: the test kits (`gitkit`, `hubforge`, `tmuxkit`, `lyxbin`), the test files that call `os.MkdirTemp`/`os.CreateTemp` with an empty or system temp dir, `internal/gatecli`'s prebuild and `cmd/testtiming`.
- A scan test that fails a bare system-temp creation outside the temp package.
- A `TestMain` calling `tmuxkit.Main` in every test package that has none, and `cmd/lyx`'s tmux-isolation scan extended to require it (D4).
- A `cleanup` verb on batten and orch, and a new top-level `lyx cleanup` that calls batten cleanup, board prune, orch cleanup, its own `.scratch/` segment, logger's retention sweep and the temp root's sweep.
- Batten's done-slug refusal names `lyx batten cleanup --apply` for the run directory.
- Docs in the same commit: a new `PATTERN.md` entry, `pattern/PATTERN-leaf-packages.md`, `pattern/PATTERN-test-isolation.md`'s "Private temp directory and leftovers", `internal/orchengine/doc.go`'s handoffs line, `doc.go` of each new or touched package (`tmuxkit` included), and `docs/overview.md`'s module table.

Out:

- `lyx fabric prune` and `lyx fabric cleanup` (D8), and remote branch deletion of any kind.
- `.lyx/shuttle/`, `.lyx/reed*`, `.lyx/config`, `.lyx/gate`, `.lyx/hub`, `.lyx/webster`, `.lyx/loom`, `.lyx/reviews`, `.lyx/batten/` (no module writes or declares it), orch notices, state and watch log, and loose top-level `.lyx` files no module declares.
- Legacy leftovers created directly in the system temp directory by code from before this task; the operator removes those once by hand.
- `os.TempDir()` calls that create nothing (`os.Chdir(os.TempDir())`, path comparisons).
- Task pairs' own `.lyx`/`.scratch`: they go with the pair at teardown.

## Decisions

### D1 — One lyx temp root with locked per-process run directories

A new leaf package (working name `internal/lyxtemp`) importing only the standard library and `internal/lock` owns a per-user root directory in the system temp directory.
The root must be a real directory, not a link, and, off Windows, with mode 0700 and owned by the current user; on Windows only the real-directory (not a link or reparse point) check applies; otherwise the package returns an error naming the path, and the caller fails rather than falling back to the bare system temp directory.
A process gets one run directory under the root on first use, named with the package's run-directory prefix; the package creates it, then takes an exclusive `internal/lock` lock on a lock file inside it, and holds the lock for the process's life.
The owner takes its lock with a blocking acquire, so a sweeper's momentary hold delays it rather than failing it.
The owner removes its run directory explicitly, unlocking first, when it ends normally.
`pattern/PATTERN-leaf-packages.md` gains a section for the package (standard library, `internal/lock`, and `gofrs/flock` through it).
Rationale: an OS file lock is released when its holder dies, so liveness is exact on every platform, and a long gate run is never taken for a leftover.

### D2 — Sweep at every start, guarded by name, lock and a creation grace

Creating a run directory first sweeps the root.
The sweep considers only direct children of the root that carry the run-directory prefix and are directories, not links.
Within the creation grace of 10 minutes such an entry is skipped untouched.
Past it, an entry with no lock file is removed, and an entry with one is removed when its lock can be taken (owner dead); the sweep never creates a lock file.
Age is the directory's modification time against an injectable clock.
An entry that vanishes under a concurrent sweeper is not an error; an entry it cannot lock or remove is reported to the caller and left.
Bound: the sweep deletes only a prefixed run directory whose owner holds no lock; it never reads inside a live run directory, never touches an unprefixed entry, and the grace covers the window between `mkdir` and taking the lock.
Rationale: the board's "swept by age at the next start", made safe for concurrent processes and long runs.

### D3 — A child inheriting a run directory works beside it

When a process inherits a `TMPDIR` that is a run directory under a lyx root (a test helper re-exec, a `lyx` binary a test runs), the package takes that run directory's root as its own, after the same root checks D1 applies on its platform, failing the same way when they fail, and creates the child's run directory beside the parent's, not inside it, as `tmuxkit`'s `kitBase` does today.
That detection moves from `tmuxkit` into the temp package.
Rationale: socket paths stay as short as today under `sun_path`, and by D2 no child ever sweeps inside its parent's live run directory.

### D4 — Test binaries use the process run directory end to end

- `gitkit.HermeticGitEnv` writes its neutral gitconfig into the process run directory instead of a bare system temp file, and `TestMain` still calls it first.
- `tmuxkit.Main` uses the process run directory as its private directory (the `TMPDIR`/`TMUX_TMPDIR` target) instead of its own `os.MkdirTemp`, keeps its socket check, sweep and `/proc` leftover scan, and removes the run directory after `m.Run` on every platform, Windows included.
  On Windows it sets `TMPDIR`, `TMP` and `TEMP` to the run directory as `setEnv` does today.
  A removal that fails, such as one a lingering child's cwd blocks on Windows, is reported on stderr and does not change the exit code; the next start's sweep removes the directory once its lock is free.
  Once-per-binary fixtures (`gitkit` repo template, `hubforge` bare and shape templates, `lyxbin` builds, `t.TempDir` trees) go with it.
- A `TestMain` that does not call `tmuxkit.Main` removes the process run directory after `m.Run` through the temp package.
- A test package with no `TestMain` gets one calling `tmuxkit.Main`, so its `t.TempDir` trees live in the process run directory; `cmd/lyx`'s tmux-isolation scan fails a test package without a `TestMain` that calls `tmuxkit.Main`, except its allowlist entries.
  An entry names one package with a reason naming why it cannot call `tmuxkit.Main`, and that package's `TestMain` removes the process run directory as the next bullet says; what an entry lets through is that package's tmux isolation and its `t.TempDir` trees left in the bare system temp directory after a kill.
- `internal/gitkit`'s closed import set in `PATTERN-leaf-packages` gains the temp package (and `gofrs/flock` transitively, which `lyxcwd` then also links).
- Departure from the board's "including on test failure and panic": a test failure is covered, but a panicked or timed-out test binary skips `TestMain`'s remaining code in Go, so its run directory is removed by the next start's sweep instead.
Rationale: the live leaks are the gitconfig created before `tmuxkit.Main` redirects `TMPDIR`, and the directories of killed binaries; one per-process directory covers both.

### D5 — Production temp users move to the root

`internal/gatecli`'s prebuilt-`lyx` directory and `cmd/testtiming`'s directories are created under the process run directory and keep their existing removal on normal exit.
Bound: after a SIGKILL of `lyx gate test`, the sweep may remove the prebuilt binary 10 minutes later; on Linux its package children already die with it through the gate's parent-death signal.
Rationale: a killed `lyx gate test` leaves a 34 MB binary, which the sweep now reclaims.

### D6 — Scan for bare system-temp creation

A `go test` scan over the AST of every Go file, production and test, fails a call to `os.MkdirTemp` or `os.CreateTemp` whose dir argument is `""` or `os.TempDir()`, outside the temp package.
A site that cannot move is admitted by an allowlist entry naming one file and the creation call in it, with a reason naming the obstacle; an admitted site still removes what it creates on normal exit, and what it lets through is that one site's directory left in the bare system temp directory after a kill.
The allowlist lives in the scan test, so every entry is a reviewed code change.
`t.TempDir()` and dir arguments naming another directory stay allowed.
A new `PATTERN.md` entry under "Paths and geometry" states the rule and names the scan.
Rationale: the board's "a scan or test that fails a new bare `os.MkdirTemp(\"\", ...)`"; banning the bare form is simpler to check than pairing each site with its cleanup, and the root makes every creation reclaimable.

### D7 — `lyx cleanup`: a top-level verb that only calls module cleanups

- New cobra module `cleanup` under the root in `cmd/lyx/main.go`, audience operator, JSON envelope.
- Default is a dry run listing, per segment, what `--apply` would remove, with `"applied": false`; `--apply` removes it.
  The dry run and `--apply` compute the candidate set with the same function per segment.
- Runs from the hub's prime worktree only; elsewhere it refuses, naming "run it from the prime worktree".
  Batten and the cleanup module are not bound by `PATTERN-refusal-way-forward`, so no `refusal-spec.md` row is added.
- Segments, in order, each calling the owning module's function in-process:
  1. batten cleanup (D9);
  2. board prune (existing `lyx board prune` behaviour);
  3. orch cleanup (D10);
  4. `.scratch/` (D11);
  5. logger's retention `Sweep` over the prime's and `_board`'s `.lyx/logs`, with the bounds the exit sweep picks (`logger.yaml` through `loggerconfig`, defaults when unusable);
  6. the temp root's sweep (D2).
- A failing segment is reported in its own result and does not stop later segments; the verb exits non-zero when any segment failed.
- Bound: `lyx cleanup` removes nothing a segment's own verb would refuse to remove; every refusal and skip of batten cleanup, board prune, orch cleanup and the `.scratch` segment lives in the module function both the verb and `lyx cleanup` call, and the verb adds only flag parsing and output.
  It adds no `--force`, and touches no remote except the records and board pushes batten cleanup and board prune already make.
  No landing or batten row calls it; it runs when the operator, or the orch on the operator's word, runs it.
- `PATTERN-sandbox-coverage`: the sandbox suite exercises the `cleanup` module, a dry run at least.
Rationale: the board's "only calls each module's own cleanup verb, so no module learns about another's state".

### D8 — Fabric's prune and cleanup stay outside `lyx cleanup`

`lyx cleanup` does not call `lyx fabric prune` or `lyx fabric cleanup`.
Rationale: their debris (a pair whose warp is gone, a weft branch whose warp sibling is gone) is also the shape of a pair that batten's Worktree-Teardown has half removed and will finish on resume through `pairteardown`, which archives the weft tip and settles the board claim; neither fabric verb shares a lock with that teardown, so running them on a schedule could pre-empt it.
They stay operator verbs for repairing hand-made damage.

### D9 — `lyx batten cleanup`

- New batten verb with the same dry-run/`--apply` shape, prime only.
- A slug is a candidate when its persisted status is `done` and its run lock can be taken; `running`, `paused`, `blocked`, `awaiting` and `failed` are never touched, a held run lock skips the slug, and an undecodable status skips it; each skip is reported with its reason.
- For each candidate, holding the run lock, it removes the records run directory (`shedrun.RunDir`) and commits the deletion through fabric's records commit path, pushing records only (`PATTERN-fabric-git`, `PATTERN-push-both-sides`).
  Files are removed with plain file removal and staged through `fabricengine`, so `PATTERN-fabric-destruction-chokepoint` (fabric's own destructive primitives) is not involved.
- A slug whose records run directory is deleted on disk but still tracked is a candidate only when the `status.json` in the records HEAD tree decodes to `done` and the run lock can be taken; otherwise it is skipped and reported, so an apply interrupted between removal and commit is finished by the next run and no other on-disk deletion is committed.
  Such a slug's scratch is never emptied by the orphan rule.
- Then, still holding the run lock and also holding the slug's status lock (`shedrun.StatusLock`) from before it decides the slug is a candidate (the on-disk status read, or the records HEAD tree read for a tracked-but-deleted run directory) until the commit is made, it removes the slug's `.lyx` run scratch files except every lock file (`run.lock`, `status.json.lock`, `last-commit.lock` and any other `*.lock`), and releases both locks; lock files and their directory stay, so no lock is deleted while held or recreated beside a held one.
- An orphan is a `.lyx` run scratch directory whose records run directory is absent on disk and absent from the records HEAD tree, whose run lock (`shedrun.RunLock`) can be taken, and whose newest modification is older than one day; it is emptied the same way, every lock file kept.
  A records run directory counts as absent only on a not-exist error; any other stat error skips the slug and is reported.
- Paths come from `internal/shedrun` (`PATTERN-shed-run-directory`).
- A push failure is reported; the commit stays for the next records sync.
- The done-slug refusal in `battencli` names `lyx batten cleanup --apply` for the run directory and keeps its advice on the leftover task branch.
Bound: it deletes only a done slug's records run directory and the scratch contents beside its lock files, or an aged orphan's scratch contents; it never touches a pair, a branch, a lock file or another slug.
Rationale: board's "removes the status directory of every run whose status is terminal and done", and the `batten-reused-slug` friction.

### D10 — `lyx orch cleanup`

New orch verb, same shape: removes every file under the orch's handoffs directory except the one the persisted `State.LastHandoff` names; with `LastHandoff` empty or naming a missing file it removes nothing and says so.
It holds the orch state write lock (`Paths.StateLockPath`) from reading the persisted cycle phase and `LastHandoff` through its last removal, so the watcher cannot enter handoff-requested in between, and it does nothing, and says so, unless that phase is idle, since a handoff is being written or read in the other phases.
`internal/orchengine/doc.go`'s "handoffs/: one timestamped handoff file per cycle, all kept" changes to state that the handoff `LastHandoff` names is kept and the rest are removed by `lyx orch cleanup`.
Bound: the file `State.LastHandoff` names always survives, so a restart still resumes from it, and a handoff a cycle starts while cleanup runs is never removed; notices, state, locks, the note template and the watch log are untouched.
Rationale: board's "old orch notes beyond the latest", read as handoff notes.

### D11 — `.scratch/` segment, owned by the cleanup module

The `cleanup` module owns the prime's `.scratch/` segment, since no module owns that directory.
A direct child is removed when the newest modification time anywhere inside it is older than 14 days and no board entry that is not done cites it by its `.scratch/<name>` path.
A board that cannot be read fails the segment and removes nothing.
A link directly under `.scratch/` is removed as a link by the same age rule, never followed.
A missing `.scratch/` is an empty segment.
Bound: only direct children of the prime's `.scratch/`, and only with the board read.
Rationale: board's "stale drafts"; 14 days matches logger's default retention age, and the citation check keeps evidence that open board entries point at.

## Constraints

- The run-directory and root names stay short enough that `tmuxkit`'s deepest socket path still passes `checkSocketPath`.
- Windows: a locked file cannot be removed, so the owner unlocks before removing its run directory, and the sweep releases a lock it took before removing.
- `PATTERN-testkit`: the temp package is not a kit and is importable from production; no kit gains an import the testkit ban list forbids.
- `PATTERN-lyxdirs-single-declarer` and `PATTERN-durable-vs-ephemeral-state`: `.lyx`/`_lyx` paths come from `lyxdirs` and the owning modules' accessors.

## Auto-mode assumptions

- The 10-minute creation grace, the one-day orphan-scratch age and the 14-day `.scratch` age are picked values, not operator-given.
- The board's "every cleanup there is" is read as its enumerated cleanups plus logger retention; D8 says why fabric's verbs are not among them.
- The board's "old orch notes" is read as the handoff notes.

## Open risks

- The operator may want fabric's prune and cleanup in `lyx cleanup` despite D8.
- `reedengine`'s socket-file tests use short `os.MkdirTemp("", "rg"|"rs")` paths for socket length and may need an allowlist entry; `reed-socket-test-fd-sweep` reworks the same tests.
- Worktrees branched before this lands keep leaking the old way until they rebase or finish.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - d10-cleanup-not-atomic-against-watcher (class design, severity MEDIUM, open in rounds 2)
  - d9-orphan-rule-overlaps-tracked-but-deleted (class consistency, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- A run directory is created under the root with its lock held; normal close removes it.
- A root that is a link is an error on every platform; off Windows, a root with another owner or not 0700 is too; each error names the path and nothing is created in the bare system temp directory.
- The sweep removes a dead owner's run directory past the grace and a lock-less prefixed one past the grace; keeps a live owner's at any age, a dead owner's within the grace and every unprefixed entry; tolerates a concurrent sweeper removing the same entry; follows no link.
- A child inheriting a run directory creates its own beside it, and its sweep leaves the parent's directory and contents alone.
- A test binary under `tmuxkit.Main` leaves nothing in the system temp directory outside the root after a passing run and after a failing test, on Linux and on Windows; its gitconfig is gone after the run.
- A killed test binary's run directory is removed by the next start's sweep.
- A run directory whose removal a lingering child blocks is reported, leaves the exit code unchanged and is removed by the next start's sweep.
- A test package with no `TestMain` of its own is refused by the tmux-isolation scan.
- The scan fails a bare `os.MkdirTemp("")`, `os.CreateTemp("")` or either with `os.TempDir()` as dir outside the temp package, ignores a mention in a comment, and passes on the tree.
- `lyx batten cleanup`: a done slug's records run directory goes in one pushed records commit and its scratch contents go beside its surviving lock files; every non-done state, a held run lock and an undecodable status are skipped and reported; an aged orphan scratch directory is emptied and a fresh one kept; an apply interrupted after removal and before commit is finished by the next run; a dry run changes nothing and lists exactly what `--apply` removes; a push failure is reported.
- A `lyx batten run <slug>` started while `lyx batten cleanup --apply` holds the slug neither re-seeds `status.json` before the deletion commit nor locks a new status-lock inode.
- A tracked-but-deleted run directory whose committed status is not done is skipped and reported, its deletion left uncommitted, and its scratch keeps its contents.
- After `lyx batten cleanup --apply`, `lyx batten run <slug>` for that slug no longer refuses on a done status.
- The done-slug refusal names `lyx batten cleanup --apply`.
- `lyx orch cleanup` keeps the handoff `State.LastHandoff` names and removes the rest, including a newer partial handoff an aborted cycle left; with no `LastHandoff` it removes nothing; it does nothing outside the idle phase; an empty or missing handoffs directory is a no-op.
- A cycle entering handoff-requested during `lyx orch cleanup --apply` loses no handoff.
- `lyx cleanup`'s orch segment outside the idle phase removes nothing, as `lyx orch cleanup` does.
- The `.scratch/` segment removes a child whose newest inner modification is older than 14 days, keeps a newer one and an old one cited by a not-done board entry, removes an old link without following it; a missing `.scratch/` is a no-op; a board read failure fails the segment with nothing removed.
- The log segment sweeps with `logger.yaml`'s bounds.
- `lyx cleanup`: a dry run lists every segment and changes nothing; `--apply` runs every segment; one failing segment does not stop the others and the exit is non-zero; outside the prime it refuses naming the prime.

## Notes for the plan writer

- Dry-run/`--apply` shapes to mirror: `lyx board prune`, `lyx fabric cleanup`.
- `cmd/lyx/exitsweep.go`'s `exitSweepBounds` is the bound choice the log segment reuses.
- `cmd/lyx/hermeticenv_test.go` and `cmd/lyx/tmuxisolation_test.go` are the existing scan tests to model the new one on.
- The `loomcli-smoke-lyx-*` creator was removed in `b7d4fd0ba`; there is no site left to fix.
- New top-level tests may push a package past `cmd/lyx/testdata/test-budget.yaml`; list it under `Edit:` when they do.
