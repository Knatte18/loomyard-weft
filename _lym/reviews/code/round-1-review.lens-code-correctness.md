```yaml
lens: code-correctness
outcome: reported
```

**Lens: code-correctness. Each function or branch the diff changes, the case I ran, and whether it is correct.**

1. `commitWeftAt`, new seed call:
   - **`SkipGit: true`.** The early return comes before the seed. Nothing is written and `("", false, nil)` comes back. Correct.
   - **Repo with no `info/` directory.** `resolveGitExcludePath` joins `CommonDir()/info/exclude`. `mutateGitExclude` runs `MkdirAll(info)` and takes its flock. It reads the file, treats a missing one as `""`, appends the five entries and writes atomically. Correct.
   - **Linked worktree.** The path comes from `CommonDir()`, so one seed covers every worktree of the repo. Correct.
   - **Second and later `Commit` calls.** The line-exact check finds every entry present, so `rewrite` returns its input and `updated == existing`. No write happens. Idempotent and correct.
   - **Content with no trailing newline.** A `"\n"` is added before the next entry. Correct.
   - **Seed fails** (`CommonDir` error, `MkdirAll`, lock, or write). It returns `("", false, err)` before any staging, so the index and worktree are untouched and the failure is clean. The error is `commitWeftAt`'s `err` result, which the `:=` in the `if` shadows. The `return "", false, err` is explicit, so this is correct. The seed's own error is already wrapped as `fabricengine: seed weft artifact excludes in %s: %w`. The wrapping is consistent with its sibling (`ensureWeftLockDirAt` returns the same error bare), so there is nothing to flag.
   - **Callers other than `Bolt`.** `commitWeftAt` is called only from `Bolt.Commit` (`bolt.go:38`). The other callers I found, such as `fabriccli/clone.go:77` and `orchcli/rules.go:70`, reach it through `Bolt`. No other caller's behavior changes. Correct.
   - **Held lock.** `commitWeftAt` takes no weft lock. The exclude flock is separate and taken and released inside `mutateGitExclude`, so there is no deadlock with a caller holding the records write lock. Correct.
   - **Exclude entries versus tracked files.** The `.lym/` exclude only affects untracked files. A board that already tracks `.lym` content keeps it tracked, which is a pre-existing limit. Not a defect for this diff.

2. `Bolt.Commit` is a doc comment only. Its claim matches the code. Correct.

3. `CloneHub` comment rewrite. Before `CloneHub` runs `seedWeftArtifactExcludes(boardDir)`, a failure goes through `teardownHub`, which is unchanged. The new text says "the board's stage-all commit seeds them again before staging", which is true. The stated ordering reason, that the seed comes first so the board never shows `.lym` as untracked dirt, is also true. The old text's reason, that Bolt seeds nothing, was falsified by D2. The new fatal-failure rationale matches the code, which returns after `teardownHub`. Correct.

4. Bypass-push step, `RunCLIIn(h.BoardDir(), …)`:
   - **Hub resolution.** `CwdFrom(ctx)` returns the seeded board directory. `lymcwd.ResolveWorktree(boardDir)` gives a `HubPath` equal to `h.Path`. This is the same as the real push child's start cwd, per the code comment in `weft_verbs.go`. Correct.
   - **Sink override and cleanup.** `SetDurableSinkDir(HubLogsDir(hub))` sets the override. Cleanup resets it with `SetDurableSinkDir("")`. The cleanup is registered before the call, so it runs on every path, including a failing step.
   - **Subtest cleanup timing.** The steps are serial and not parallel, and the scenario itself is parallel. `t.Cleanup` on the step's `t` fires when that subtest ends. The sink is process-wide, and other parallel tests in the binary could observe it between the call and the cleanup. That exposure is narrower than before, since the sink now points at the test hub's own logs, which are removed with the hub. Not a correctness defect.
   - **Assertion.** `logger.TraceDir()` returns the override directory when set (`resolveSinkDirLocked` is not read here, but `TraceDir` is documented as the directory the sink would write to). It checks the post-condition `HubLogsDir(h.Path)`, which is the scenario's goal and not the setup. A parallel test calling `SetDurableSinkDir` in the same binary could race with it. I found none in this diff. Not a finding.
   - **`resolveSinkDirLocked` override precedence.** I did not read it. I assumed `sinkDirOverride` wins. This is unverified, but the assertion itself would fail loudly if wrong.

5. `TestBolt_DirtyRepo_CommitsAndPushes`:
   - **Fixture.** An unseeded fixture holds `bolt-note.md` plus `.lym/logs/trace.log`.
   - **Commit.** `Commit` seeds the excludes, then stages. `ls-files` should be exactly `["bolt-note.md"]`.
   - **A `.gitignore` or other tracked seed file in the fixture.** `newBoltRepo` was not read. If it commits a seed file, the `ls-files` equality would fail. This is unverified, and the verify gate would catch it.

6. `TestBolt_CleanRepo_CommitAndPushAreNoOps`:
   - **Push lock.** `Push` leaves `.gitrepo-push.lock` at the repo root. `gitrepo.PushLockFileName` is in the seeded entries (`recordsgit.go:111`). The next `Commit` seeds first, so the lock stays untracked and the call reports `committed=false`. The removal was correctly dropped.
   - **`*.lock`.** `lockFilePattern` also covers it.

**Findings:** none. Nothing wrong in the changed code.

**Tool calls (1):**
1. `sed -n 1,51p internal/fabricengine/gitexclude.go`: I needed `resolveGitExcludePath` and `mutateGitExclude`'s preamble to confirm the no-`info/` and common-gitdir behavior. My context had only the second half of `mutateGitExclude`.
