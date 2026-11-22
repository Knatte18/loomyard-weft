```yaml
lens: code-tests
outcome: reported
```

Lens code-tests: which test row fails when each behaviour the diff adds or changes breaks.

**Behaviours and the tests that pin them:**
- **Mutating verb passes when cwd is in a hub under the strand's own sandbox root:** `TestSandboxedRoleGuard`, row `OwnSandboxHub`, over every mutating form (`internal/fabriccli/roleguard_test.go:263`). Pinned.
- **Refused in another worktree's sandbox hub (the `WorktreeName == slug` check):** row `OtherWorktreesSandboxHub` (`roleguard_test.go:264`). Pinned.
- **Refused under a lookalike sandbox root whose holder is not a git worktree (the `err == nil` check):** row `SandboxRootOfADirectoryNoWorktree` (`roleguard_test.go:265`). Pinned.
- **Refused in the strand's own worktree outside its sandbox root:** row `OwnWorktreeOutsideItsSandboxRoot` (`roleguard_test.go:266`). Pinned.
- **Refused rows exit 1:** the `code != 1` assertion (`roleguard_test.go:274`). Pinned.
- **`verbTargetDir` judges clone's absolute `--into` instead of the cwd:** clone rows `IntoOwnSandboxRootFromItsWorktree` passes and `IntoAnotherDirectoryFromOwnSandboxHub` is refused (`roleguard_test.go:286-287`). Pinned.
- **A clone destination that does not exist yet is compared as written:** row `IntoOwnSandboxRootFromItsWorktree`, since `hub-2` is never created (`roleguard_test.go:286`). Pinned.
- **`verbTargetDir` resolves a relative `--into` against the cwd:** none.
- **`inOwnSandboxRoot` resolves symlinks in an existing dir:** none. No row uses a symlinked path.
- **`inOwnSandboxRoot` requires `location.WorktreePath() == worktree`:** none. No row puts a sandbox root under a subdirectory of a worktree named after the slug.
- **The nearest enclosing sandbox root decides:** none. This is a narrow nesting case and no finding is raised for it.
- **The sandbox pass sits below the path-flag refusal and does not widen it:** none. The path-flag rows of `TestSandboxedRoleGuard` run from an empty temp dir through `runUnderParent`, never from a sandbox hub.
- **The refusal text names `.lym/harden-sandbox`:** not asserted; only `refusalMarker` "from the task session" is checked. The new clause is prose an agent reads, so no finding is raised.
- **The `cloneDestination` extraction keeps empty and absolute `--into` working:** `TestCommand_CloneDestination` rows `WithoutIntoFlagUsesResolvedCwd` and `IntoFlagCreatesHubAtDirectory` (`internal/fabriccli/cli_test.go:230-231`). Pinned.
- **The extraction keeps relative `--into` working:** no row. The gap existed before this diff and the behaviour did not change, so no finding is raised.

**Findings:**

1. LOW, design, `internal/fabriccli/roleguard_test.go:280-288`: the guard's handling of a relative clone `--into` has no row.
   Every clone row passes an absolute `--into`.
   If `verbTargetDir` judged the raw `into` instead of `cloneDestination(cwd, into)`, `clone --into hub-2` from the own sandbox root would be refused and no test would fail.
   Evidence: `internal/fabriccli/roleguard.go:35-36`.
   Fix: add a clone row with a relative `--into`, such as `.lym/harden-sandbox/hub-3` run from `own`, expecting a pass.

2. LOW, design, `internal/fabriccli/roleguard_test.go:258-267`: two checks in `inOwnSandboxRoot` have no row that fails when they break.
   - The `location.WorktreePath() == worktree` check: a sandbox root at a subdirectory of the own worktree, such as `&lt;own&gt;/sub/.lym/harden-sandbox/hub-1`, should be refused.
     Without the check `ResolveWorktree` still reports `WorktreeName` "slug" and the row would pass.
   - Symlink resolution: a cwd reached through a link into the own sandbox hub should pass.
     Evidence: `internal/fabriccli/roleguard.go:45-53`.
   Fix: add a `OwnWorktreeSubdirSandboxHub` row expecting refusal and a `LinkIntoOwnSandboxHub` row expecting a pass.

3. LOW, design, `internal/fabriccli/roleguard_test.go` (the existing path-flag rows): no row checks that the new sandbox pass cannot let a path-flag call through.
   `sandboxedRoleRefusal` refuses `--weft-path` on any verb but push before it reaches the sandbox pass (`roleguard.go:119-133`).
   No row runs such a call from the own sandbox hub, so moving the sandbox pass above the path-flag branch would pass every test.
   Fix: add a row running, for example, `pull --weft-path /x` from `ownSandboxHub` and expect the path-flag refusal.

Tool calls:
- Grepped `TestCommand_CloneDestination` in `internal/fabriccli/cli_test.go` to see which `--into` forms the extracted `cloneDestination` is tested with; it was outside the manifest's reading.
