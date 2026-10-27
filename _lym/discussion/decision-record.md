# Decision record — hub-reader-creates-lym

## Goal

A read never creates a directory, so a hub-wide reader polling a pair that `Topology.Add` is still building can no longer leave a real `<warp>/_lym` that the junction check then refuses.

## Scope

- `state.ReadJSON` drops its `os.MkdirAll`; `ReadJSONNoMkdir` folds into it and every caller moves over.
- `fabricengine.ReadOrigin` stops creating the weft `.weft` lock directory and seeding its exclude for a read.
- `websterengine.LoadState` stops creating its scratch directory for a read.
- `orchcli.HubOrchs` skips a worktree under a `.fabric-wt-staging-*` directory and a pair whose `_lym` junction is not yet a link.
- `Topology.Add` wires the junctions right after the weft worktree exists, before the local-branch marks and the launchers.
- Out of scope: lock files a read creates inside an existing directory, which the read lock has always done.

## Decisions

### A read whose lock directory is absent reads without the lock

Decision: `ReadJSON` takes the read lock as before; when the lock file's directory does not exist, it reads the file without a lock instead of failing.
A missing data directory then reads as a missing file through the ordinary not-exist path.

Rationale: no writer can hold a lock in a directory that does not exist, since every writer creates the directory before locking, and every write replaces the file by atomic rename, so an unlocked read still sees a whole file.
This also covers a tracked record on a machine whose never-tracked lock directory was never made, such as `_lym` state with no `.lym` scratch yet.
Bound: the lock is skipped only on a not-exist error from opening the lock file; any other lock error still fails the read.

### One read, one error shape

Decision: `ReadJSON` wraps a read failure other than not-exist in `ErrRead` and a decode failure in `ErrDecode`, both with `%w`, and `ReadJSONNoMkdir` is removed.

Rationale: callers of the removed function test `errors.Is` against both sentinels; keeping both on the one reader lets every caller move over unchanged.

### ReadOrigin builds its lock path without creating it

Decision: `ReadOrigin` joins the origin lock file onto the weft worktree's `.weft` path directly; only `WriteOrigin` keeps the creating, exclude-seeding lock path.

Rationale: the seeding is a write, and it is what leaves a stray `.weft` under a staging directory when a reader polls mid-creation.

### HubOrchs skips half-built pairs before reading them

Decision: `HubOrchs` skips a worktree whose path sits under a `.fabric-wt-staging-*` directory before resolving it, and a resolved pair whose `<anchor>/_lym` is not a link before reading its origin.
fabricengine owns both checks, as an exported staging predicate and an exported junction predicate, so orchcli names no fabric path.

Rationale: a pair without its `_lym` junction has no readable origin yet, so skipping it loses nothing, and the predicates keep fabric geometry inside fabricengine.

### Add wires junctions before the launchers

Decision: in `Topology.Add`, the junction wiring moves to directly after the weft worktree is created or adopted and moved to its anchor, ahead of the local-branch marks and `writeLaunchers`.

Rationale: it shortens the window in which the warp worktree exists without its junction, and neither the marks nor the launchers depend on the order.

### Added after Discussion (parent, 2026-10-27): Leave the bench link breaks to main

Decision: Way (c): do not touch the docs/research bench files or the docs link allowlist in this run. Task docs-link-skip-bench-data fixes TestEnforcement_MarkdownLinks on main by skipping docs/research/*/data/; once it lands, the orch resumes this run and it takes main in through its normal path.

Rationale: The breaks come from main's bench commits, not this change; one fix on main serves every run.

## Constraints

- `PATTERN-fabric-git`: the new predicates live in `internal/fabricengine`; orchcli runs no git and builds no fabric path.
- `PATTERN-cwd-resolution`: `HubOrchs` keeps resolving each worktree through `lymcwd`.
- `PATTERN-tolerant-records`: `ReadJSON` keeps decoding with a plain unmarshal.
- `PATTERN-test-speed` and `PATTERN-test-isolation`: new tests carry the lowest tier that fits and use hermetic git.
- Filesystem links are checked through `internal/fslink`.

## Auto-mode assumptions

- The board entry's test, "holds `Add` between weft creation and wiring and runs `HubOrchs`", cannot hold `Add` from orchcli without an exported test seam in production code.
  The test lives in fabricengine instead, holding `Add` through an unexported seam and running the reader path `HubOrchs` takes (`List`, `lymcwd.ResolveWorktree`, `ReadOrigin`) on every listed worktree; a separate orchcli case covers `HubOrchs` skipping an unwired pair.
- A darn run holds no interview; every call here is the writer's own.

## Open risks

- A caller that wrote into a directory with a raw `os` call, relying on an earlier read to have created it, fails after this change.
  A sweep of readers found none: writers go through `WriteJSON`, `UpdateJSON` or `fsx.AtomicWriteBytes`, which create their directory, and lock-taking writers create their lock directory first.

## Acceptance criteria

- Reading a JSON record whose directory is absent returns not found and creates no directory.
- Reading a present record whose lock directory is absent returns the record and creates no directory.
- A read error other than not-exist matches `ErrRead`; a malformed file matches `ErrDecode`.
- With `Add` held between weft creation and wiring, the hub reader path run over every listed worktree leaves no `<warp>/_lym` and no `.weft` under a staging directory, and `Add` then completes.
- `HubOrchs` lists no pair whose `_lym` is not a link and logs no warning for it.
- `websterengine.LoadState` on a worktree with no scratch directory returns no state and creates nothing.
- No reference to `ReadJSONNoMkdir` remains.

## Notes for the plan writer

- `internal/state/doc.go` documents the readers and changes with `ReadJSON`.
