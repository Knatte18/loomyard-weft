```yaml
lens: code-pattern-fit
outcome: reported
```

**Lens: code-pattern-fit.** I checked the diff against `PATTERN.md`, the `pattern/` background files, `CLAUDE.md` and the commit conventions. I found no BLOCKING defect and one LOW consistency finding.

**Rules checked**
- `PATTERN-fabric-git`: held. The new predicates `InWorktreeStaging` (`destroy.go`) and `LymJunctionWiredHere` (`junction.go`) live in `fabricengine`. `orchcli/live.go` calls them and runs no git. It builds no fabric path.
- `PATTERN-cwd-resolution`: held. `HubOrchs` still resolves each worktree through `lymcwd.ResolveWorktree`. `ReadOrigin` takes absolute paths from `RecordsWorktree` and `RecordsLockDirPath`.
- `PATTERN-tolerant-records`: held. `ReadJSON` still decodes with a plain `json.Unmarshal`. Both sentinels are wrapped with `%w`.
- `PATTERN-test-speed` and `PATTERN-test-isolation`: held. Both new tests carry `//go:build tier1`, call `t.Parallel()`, and use `hubforge`, which supplies hermetic git. The hook is per-`Topology`, so it is parallel-safe.
- Filesystem links through `internal/fslink`: held. `LymJunctionWiredHere` uses `fslink.IsLink`, and both new tests use `fslink.IsLink` or `fslink.Remove`.
- `PATTERN-durable-vs-ephemeral-state`: held. No new path constants are named outside `lymdirs` and `fabricengine`. The `.fabric-wt-staging-` literal moved into `worktreeStagingPrefix` inside `fabricengine`.
- Docs in the same commit: held. `internal/state/doc.go` gains "Reads create nothing", `internal/orchcli/doc.go` gains the `HubOrchs` skip, the `add.go` header is updated, and `contracts/specs/loom-status-spec.md` is touched. `internal/fabricengine/doc.go` does not describe the Add step order, so it needs no edit.
- Declared PATTERN changes landed: raise nothing. The decision record declares no PATTERN entry, and the diff touches no `PATTERN.md` or `pattern/` file.
- Commit messages and departures: held. No commit carries a `Pattern-Departure:` trailer, and no difference from the answer key needs one.
- Comment conventions (`scribe:golang-comments`): held. New doc comments start with the symbol name, and no new end-of-line comments appear. The moved `(10b)` comment is now reflowed into sentences.
- `PATTERN-sparse-hub-base` and `PATTERN-stencil-ownership`: not touched by the diff.

**Findings**

1. **LOW, `consistency`, `internal/loomengine/seed.go:123-125`.**
   - `VerifySeedOwnership` still runs `os.MkdirAll(filepath.Dir(statusLockPath))` before `state.ReadJSON`. That is a read that creates a directory.
   - The entry says "a read never creates a directory, anywhere". The new `internal/state/doc.go` section "Reads create nothing" says the same. The commit message lists `CheckSeed`, `LoadState`, `checkRunStatus` and the status verb as the callers moved over, but not this one.
   - `ReadJSON` now reads without the lock when the lock directory is absent, so the `MkdirAll` is dead weight.
   - Fix: drop that `MkdirAll` and the `os` and `filepath` imports if nothing else needs them. If a hub poll never reaches this path, record that as the reason to keep it.

   Other `MkdirAll(Dir(lock))` calls in the grep sit before write or lock-taking paths: `shedengine/run.go`, `goto.go`, `rename.go`, `failedstop.go`, `loomcli/resume.go` and `loomcli/arm.go`. I did not verify any of them. `loomshed/seed.go:40` is the one I would look at next, but it is outside the diff, so I am not raising it.

**Tool calls**
1. One combined Bash call. It grepped `MkdirAll(filepath.Dir(...lock))` sites and `ensureStatusLockDir`, read `VerifySeedOwnership` in `loomengine/seed.go`, grepped the Add and junction wording in `fabricengine/doc.go`, and ran `git diff --stat` on `PATTERN.md`, `pattern` and `docs`. The manifest lacked the surviving pre-read mkdir sites, the fabricengine doc wording, and whether the diff touched `PATTERN.md`.
