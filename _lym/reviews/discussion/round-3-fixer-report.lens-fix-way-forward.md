```yaml
lens: fix-way-forward
outcome: reported
```

Lens fix-way-forward: no findings. Every step, test and refusal the fix introduced can be carried out from the state it names.

- **Unset-seam refusal as a core row (D13, D5, acceptance criterion):** the engine knows whether each required seam is installed before it takes any lock. So the "supplied" snapshot fact is available in the pre-lock phase D2 already defines, and a tier0 row over a snapshot that marks a seam unsupplied can be written with no process-global state.
- **Sequential seam-setting rule (D13):** fabric's setters today all return a restore: `InstallWorkspaceContent`, `InstallPairFolderNamer`, `InstallPushNotifier` and `InstallWorktreeSeam` (`internal/fabricengine/hubworkspace.go:91,115`, `failedpush.go:106`, `worktreeseam.go:50`). So "puts back what was there" works the way `internal/ideengine/spawn_hub_integration_test.go:180` already does it, through `t.Cleanup`. The "build the hub first" order also matters. `internal/hubforge` installs the required seams on the first hub build and never restores them. If a test set a seam before that build, its restore would remove hubforge's install for every later test.
- **`internal/beamcli/dobbyleftover_integration_test.go:40` calls `t.Parallel()`:** the rule covers this test. Dropping that call is a local step the test can take.
- **`internal/stencilsource` (new):** it is reachable from `main` through `internal/stencilcli` and `cmd/lym`, so the path-taking reads fabric adds have production callers. `cmd/lym/stencilseed.go`'s `stencilstore.RegisterOrdering(...StencilSource(...).Older)` keeps working once it imports the new package. D9 part 5 gives the move a home ahead of the part 7 guard.

Tool calls:
- One grep of `internal/fabricengine` setter signatures and the seam installs in the ideengine and beamcli tests. My context did not hold whether today's setters return a restore, or which of those tests call `t.Parallel`.
