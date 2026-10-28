```yaml
lens: code-affected-set
outcome: reported
```

**Task:** code-affected-set lens for the gate-robustness diff. I found no member of the affected set that the diff leaves wrong, so no findings.

**Per change, the set found:**

1. **Unset `Tier.Dir` and `Build.Dir` now stay empty until `WithDefaultDir`.** Covered.
   - Readers of the dir all run on the filled `cfg`:
     - `gatecli/build.go:32`
     - `gatecli/dotnetscope.go:69,75`
     - `gatecli/scope.go:312`
     - `gatecli/tiers.go:157,182,185`
     - `gatecli/green.go:84,359,421,432,447`
   - `WithDefaultDir` is applied in `run.go:67`, right after load and before `request.root` and the tier planning.
   - A worktree anchored at its root gets `.`, so fingerprints and green-record module keys are unchanged. `RunGate` goes through `runTest`, so it is covered too.
   - The other `LoadConfig` caller, `hubgeom/gateslots.go:22`, reads only `cfg.Limits()`, so an unfilled dir does not matter there.
   - Guard runs use the fixed `guardDir: "."` at `scope.go:218`. That is unchanged and the diff does not touch it.
   - `driftengine/config.go:176` defaults its own separate step dir and is not a gate reader.
   - Docs and templates that said "default `.`" were updated:
     - `gateslot/doc.go`
     - `template.yaml` (both places)
     - `gatecli/doc.go`
     - the `Dir` field comments in `config.go` and `tiers.go`
   - A repo-wide grep for `dir (default` found no stale mention.
   - The updated test rows cover the removal of the `.` default. `TestGateTest_UnsetDirFollowsTheAnchor` covers the anchor case for the build, tier0 and a tier6 with `dir: .`.

2. **Build-server variables.** Covered.
   - `reedengine/panebin.go` and `gatecli/run.go:128` (`withBuildServerOff`) are the only two sites. A tier's own `env` is applied later, so it can still override.
   - Test and doc updates:
     - `composePaneLaunchLine` tests in `panebin_test.go` and `spawn_test.go`.
     - The gate integration test, which sets inherited conflicting values.
     - `reedengine/doc.go:178`, `proc/doc.go`, and the header of `panebin.go`.
     - The "Warm build processes" section of `docs/dotnet-hub.md`.
     - The `gatecli/doc.go` lines about the build command.
   - `reedengine/spawn.go:74`, a comment about the exports, names only name, parent and bin. That is not wrong, only incomplete, so I am not raising it.
   - `verifytree/verify.go:382` builds its own env for the command it runs. That command is `lym gate test`, which sets the variables itself, so this is covered.
   - `OpenWindow` and `composeWindowCommand` export nothing. They run `lym` verbs, not agent sessions, so they sit outside the decision record's scope.
   - `shell.ExportEnv` is implemented for posix and pwsh, so both flavors are covered by the `sh.ExportEnv` assertions.

3. **`FailedRunsPrefix` and the failed-runs line.** Covered.
   - The only producer is `printFailedRuns`, and the only consumer is `loomshed.failedRunsLine`.
   - The `gatecli` tests assert the new label through `failedRunLabel`.
   - A grep found no other reader of the old literal `gate test: failed runs:` outside tests.

4. **The shortage retry.** Covered.
   - `runGreenAware` is the single call site. The `RunGate` and `newCommand` paths both set `shortageRetryDelay: defaultShortageRetryDelay`.
   - Every `testRequest` literal in non-test code sets the delay. Test literals leave it at 0, which is harmless.
   - `gatecli/doc.go` documents the behavior.
   - `PATTERN.md` and `pattern/` mention no shortage or retry rule. The decision record declares no PATTERN change, so none is owed.

5. **Verify findings.** Covered. `loomshed/doc.go` and the `verifyFailureFindings` comment are updated, and `verifygate_test.go` has the new row. Its `wantNot` of "Log tail:" covers the no-tail case. The existing rows cover the fallback to the tail.

6. **Commit messages.** Neither commit has a `Pattern-Departure:` trailer, and none is needed.

**Tool calls:**
1. grep for Tier/Build `.Dir` readers, `failed runs:` literals and `dir (default` mentions. My context did not hold the reader set outside the diff.
2. grep for `LoadConfig` callers, strand-env export sites and `ExportEnv` implementations. I needed other env-composition sites and the shell flavors.
3. Read `hubgeom/gateslots.go`, grepped for `Dir: "."` defaults, and grepped docs and PATTERN for shortage or retry and gate env/dir. I needed to check the remaining `LoadConfig` consumer and whether any PATTERN text was affected.
