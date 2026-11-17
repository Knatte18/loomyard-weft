```yaml
lens: guard-reach
outcome: reported
```

Guard-reach lens: per guard, reachable or not.

**Findings**

1. MEDIUM — Card 9, `TestNew_CodexLaunchCarriesTheClaudeUserSetup`: the test cannot reach its assertion as written.
   - The test writes `sandbox_mode: danger-full-access` into a codex.yaml local layer so that no probe runs.
   - `ConfigSpec` classifies `sandbox_mode` as `configengine.CategoryMachine`, and the local layer refuses every key outside `CategoryNone` (`internal/configengine/layers.go:170-171`, `LocalDenied`; the `internal/shuttleengine/codexengine#ConfigSpec` comment says "the local layer may not").
   - So `LoadConfig` refuses, and `Prepare` returns the `loadConfigOrRefuse` error before any user setup is requested. The global-file assertion is never reached.
   - Fix: write the mode in the worktree layer `configengine.ConfigFile(anchor, "codex")`, as `TestPrepare_SandboxMode` does, and keep the homes in the local layer.

2. MEDIUM — Card 8, `RunPluginHook`'s timeout: the timeout does not hold when the command's shell forks a child.
   - The card runs `sh -c <command>` under a context bounded by `Timeout` and waits with `cmd.Wait`. It sets no `cmd.WaitDelay` and no process group.
   - When the timeout fires, the context kills only `sh`. A grandchild such as the `sleep` that `sh` forks keeps the stdout pipe open, so `cmd.Wait` blocks until that grandchild exits.
   - In production, a hung plugin hook then holds `Prepare` past `plugin_hook_timeout`.
   - The planned row "a `sleep` past a short timeout returns an error within the bound" passes only where `sh` execs its last command. dash, the Ubuntu `sh`, does not reliably do that, so the row is flaky there.
   - Fix: kill the process group with `Setpgid` and a `Cancel` that signals `-pid`, and set `cmd.WaitDelay`. Make the row use a command that forks, such as `sleep 5; true`.

3. LOW — Card 1, `TestExpand`'s `~/` row: the row cannot run if its parent test is parallel.
   - The row uses `t.Setenv`, which panics in any test with a parallel ancestor.
   - The card puts the row in its own non-parallel subtest, but does not say that `TestExpand` itself stays non-parallel. A parallel parent (the package norm, and how every other new table in this plan is planned) makes the row panic.
   - Fix: state that `TestExpand` does not call `t.Parallel`, or move the `~/` row to its own top-level non-parallel test.

4. LOW — Card 10, the walk-error refusal: the refusal is reachable, but it is wider than what it guards.
   - `unreadClaudeFiles` walks the whole checkout root, and any walk or stat error other than a missing file refuses every launch.
   - One unreadable directory anywhere in the worktree reaches that state, for example a root-owned build or cache directory. The directory need not lie under the working directory, the only place a "below" CLAUDE.md counts.
   - The walk above the working directory only needs the path's directories, so a refusal for an error off that path blocks launches for state the guard does not concern.
   - Fix: walk only the working directory's subtree plus the directories on the root-to-working-directory path, or log an unreadable subdirectory at Warn and skip it.

5. LOW — Card 6, `codexCommand`'s "cap above zero" branch: no production launch reaches the zero side.
   - `LoadConfig` refuses `max_subagent_threads` below 1, and the template sets 16.
   - `Prepare` always loads codex.yaml, so its line always carries the cap.
   - Only `OpenLaunch` (zero by construction) and hand-built test lines reach the omit side. That is harmless, but the "else omitted" behaviour is not a reachable production state for `Prepare`.

Every other guard is reachable:
- **Card 2:** the `Show` refusal, reached by `lym heddle show agents-prompt`, which calls `renderer.Show` directly.
- **Card 4:** both sides of the empty-`Root` branch (the fixture request versus `requestOf` and `renderInto`), and the record match in `TookIn`. Its root-relative paths are consistent with `ev.Paths`.
- **Card 3:** the walk render. The heddle seam is installed in `cmd/lym/main.go:173` on the same path as the start verbs.
- **Card 5:** the cap refusal, reached by an oversized passage.
- **Card 6:** the unlisted-effort warning, reached by `xhigh`.
- **Card 8:** the matcher filter, the nil-runner omission and the failing-hook omission.
- **Card 9:** the source-error refusal, and the session id. The run directory's base name is the run id (`internal/shuttleengine/rundir.go:183-188`).
- The 128 KiB bound is not undercut by tmux: reed types a source statement for a launch script, not the line itself (`internal/reedengine/spawn.go:77,134`).

Calls made:
- `cmd/lym/main.go:160-190` and a grep of `configengine` for `LocalDenied`: to confirm where the worktree seam is installed and whether the local layer refuses a non-None key. The context lacked both.
- `internal/shuttleengine/rundir.go` and a grep of `reedengine` for `send-keys`: to confirm that the run directory's base name is the run id, and how a launch line reaches the pane. The context lacked both.
