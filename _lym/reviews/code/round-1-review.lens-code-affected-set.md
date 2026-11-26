```yaml
lens: code-affected-set
outcome: reported
```

Task: review the code-affected-set lens. For each change in the diff, I traced what calls it, reads it or asserts it, and checked the repo's rules against it.

**Per change: the affected set, and whether it is covered**

1. **`waitForFixtureProcessesToExit` now takes `(t, patience)`** (`internal/loomcli/smoke_helpers_test.go:283`).
   - Callers are `smoke_prebootstrap_integration_test.go:36`, `smoke_prebootstrap_integration_test.go:127`, `smoke_starttail_test.go:121`, `smoke_helpers_test.go:259` and `fixtureprocesses_integration_linux_test.go:27`.
   - Every caller passes a patience: covered.
   - No doc, stencil, spec or `pattern/` file names the function. A grep over `.md`, `.go` and `.yaml` outside the diff found no other hit: covered.
2. **The cleanup now kills a lingering process and fails the test when a killed one survives.**
   - The `smoke_starttail_test.go:119-120` comment says the wait is registered before the teardown so it runs after the teardown's kills. That still holds under the new kill path: covered.
   - Under `PATTERN-child-processes`, test-fixture machinery is outside the spawn rule. `proc.KillPID` was already used by the fixture teardown in `smoke_test.go`: covered.
   - Under `PATTERN-test-speed` ("Time is injectable"), the patience is a parameter and not a package `var`, and the test passes 100 ms: covered.
3. **New helper `processesUnder`.** It uses the spawn-free `tmuxkit` `/proc` probes and lives in a support file, which holds no test function. The tier-purity scan does not read support files: covered.
4. **`smoke_helpers_test.go` now imports `internal/proc`.** `smoke_test.go`, in the same package's tests, already imports it, and no kit import rule applies to a `_test.go` file of `loomcli`: covered.
5. **`newBadReedUpFixture` registers the wait itself, and `TestLoomResumeReconcilesTheHubConfigBeforeArming` drops its own call.**
   - Callers are `smoke_prebootstrap_integration_test.go:175, 266, 295, 303`.
   - Registration order is unchanged for the resume test: right after `CopyRegistered`, whose `TempDir` removal was registered first, so the wait still runs before the removal.
   - The new doc comment on `newBadReedUpFixture` matches what it now does: covered.
6. **New test file `fixtureprocesses_integration_linux_test.go`.**
   - Its tier1 functions open with `tiergate.Require(t, 1)`, it has no tier build constraint, and the `_linux` suffix is the OS-only placement `PATTERN-test-speed` ("No test skips by design") asks for.
   - It references no `tmuxkit` export itself, so the tier1 tmuxkit-spawn rule is not triggered.
   - Its file header is present: covered.
7. **Commit messages and departures.**
   - Commit `15ab7f4ba` carries a `Root-Cause:` trailer naming the defect the diff removes: the silent give-up and the missing registration in `newBadReedUpFixture`.
   - There is no `Pattern-Departure:` and no `Loosened-Check:` trailer. None is needed: the diff only tightens the cleanup, which used to give up silently, and removes no assertion.
   - Stencil wording: none changed. Declared PATTERN changes: the directive declares none.

**Findings**

None.

**Tool calls**

1. Grepped `.md`, `.go` and `.yaml` across the repo for `waitForFixtureProcessesToExit`, `fixtureprocesses` and `processesUnder`, and grepped `cmd/lym/tiertags_test.go` and `cmd/lym/tierpurity_test.go` for filename rules and `loomcli` allowlist entries. My context had only the `internal/` caller grep, not docs or the guard files.
