```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-consistency lens for the F1–F3 change set.

No findings.

- The PATTERN sentence matches `internal/testkit/tmuxkit/tmuxkit.go:461`, which sets `GIT_CEILING_DIRECTORIES` to the run directory unconditionally, and the tmuxkit tests at lines 52 and 62 check it.
- The new serial test sets the variable to the normalized ceiling. `gitCeilingDirectories` normalizes entries the same way, so the two agree.
- The header comment of `TestRepositoryRootOf` ("driven through the unexported walk so the test stays parallel") stays true.
- The new serial test's doc comment says it is serial because of `t.Setenv`.
- The file header at `internal/cliwire/cliwire_test.go:5` says only the ResolveStandalone tests stay serial, and the new serial test is not a ResolveStandalone test. This is stale only in a minor way, so I am not raising it as a finding.

Calls made:
1. `grep -rn GIT_CEILING_DIRECTORIES` over `.go` and `.md` files, to confirm the PATTERN sentence matches what `tmuxkit.Main` does. My context lacked `tmuxkit.go`.
