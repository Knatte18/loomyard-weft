```yaml
lens: code-tests
outcome: reported
```

**Behaviour → pinning test**

1. `selfexec.LymBinary` refuses the test binary: `selfexec/binary_test.go` `TestLymBinary_RefusesTheRunningTestBinary`.
2. `selfexec.ResolveLymBinary` (stand-in passes, lookup error propagates, test-binary path refused): `selfexec/binary_test.go` `TestResolveLymBinary`.
   - Not pinned: the branch where `os.Executable()` fails under test, which refuses every path.
3. Reed pins refuse the test binary: `reedengine/bindings_test.go` `TestPinBindingsLocked_PinsTheSlackLinesOnlyWhenEverythingResolves`.
   - The `TestBinaryExecutable` row now wants no button line, which covers the Slack toggle, the Slack state and the button.
   - The switch and follow bindings are pinned in the same `pinBindingsLocked` call, but the row asserts only the toggle and button lines. A switch or follow line that dropped back to the raw lookup would not fail it.
   - The pane prelude, the window command and the watchdog spawn (`composePaneLaunchLine`, `composeWindowCommand`) have no row for the refusal.
   - `buttonclick.go`'s `deps.executablePath` default is now `selfexec.LymBinary`. No test hands it a test binary and expects a refused button prompt re-entry.
4. loom's `loopExecutable` (`c.lymBinary`, default `selfexec.LymBinary`):
   - `cli_test.go` `LoopArmedForALocatedReceiver` covers the stand-in path.
   - Nothing covers the default, or the Warn and empty path when the lookup refuses.
   - `looparm_test.go` passes `""`, which covers only registration.
5. `loomcli` `spawnGoDriver`, orch's `c.executable = selfexec.LymBinary`, `adoptcli` `startSuite`/`resumeChild`, `battencli`, `fabricengine`, `driftcli` and `fabriccli`: none. Each swaps the lookup with no test of refusal.
   - The `testing.Testing()` guards in `adoptcli` and `orchcli` already return before the lookup, so the change is unreachable under test.
6. `fronttask.startAction` takes an injectable lookup: `fronttask/hub_test.go` `TestStartAction` covers the stand-in path only. The refusal path (lookup error returns "locate the lym executable") has no row.
7. `tmuxkit` refusal of a positional first argument (`refusesReexec`): `tmuxkit/reexecguard_test.go` `TestRefusesReexec`.
   - Rows cover none, flags only, a two-token flag value and a lym verb.
   - Not pinned: a lone positional argument, or a flag-looking first word followed by a positional one.
8. `Main` returns 2 and runs no suite: `reexecguard_integration_test.go`.
   - This exercises the real `Main` end to end, including the exit status and an empty stdout.
   - `TestRefusesReexec` and this test overlap in the `slack bar` case, but they pin different layers. I raise no redundancy finding.
9. The guard in every package: `cmd/lym/tmuxisolation_test.go` enforces `tmuxkit.Main` with an empty allowlist. This covers decision item 3 by construction.
10. The `gitkit` `refuseCLIReexec` removal: the old `gitkit/reexecguard_test.go` was deleted and its coverage moved to `tmuxkit`. No gap.
11. `proc.PIDsWithEnv` on Linux (matches a whole entry, skips a prefix lookalike): `proc/envmarker_linux_test.go`.
    - Not pinned: an unreadable `/proc` returning an error, and a process with an unreadable environ being skipped.
12. `PIDsWithEnv` on Windows returns `ErrUnsupported`: none. The file has no test, but it is a one-line stub.
13. `processCap.run` decisions (at the cap kills nothing, over the cap kills now and later, a failed list is skipped, an unsupported platform ends the cap): `processcap_test.go` `TestProcessCap`.
14. `runMarker`: `processcap_test.go` `TestRunMarker`.
15. `newProcessCap` production wiring (`proc.PIDsWithEnv`, SIGKILL via `proc.SignalProcess`, `time.After`, limit 1024): none.
16. The `tierrun.go` wiring, the main "a runaway fails the gate" behaviour:
    - A non-zero exit is forced when the command exited 0 with `capLine` set.
    - The cap goroutine starts only when `marker != "" && samplePeriod > 0`, and `capLine` is shared with it.
    - `runInvocations` stops on `capLine != ""`.
    - `failedOnShortage` returns "" when `processCapped`.
    - `tierResult.processCapped` is set.
    - No test pins any of these. `processcap_test.go` covers only the cap in isolation.

**Findings**

- **MEDIUM, design (behaviour with no test row).** Location: `internal/gatecli/tierrun.go` (the `runTier` cap block, `runInvocations`, `failedOnShortage`). The wiring that turns a tripped cap into a failed gate has no test. The unpinned parts are the forced exit 1, the no-rerun rule through `failedOnShortage`'s new `processCapped` clause, the `capLine` early return, and the guard on `marker`/`samplePeriod`. This is decision item 4's guarantee that "a runaway loop fails the gate". Evidence: `processcap_test.go:54-99` and `:101-121` are the only cap tests, and the grep for `processCapped` in tests found nothing. A test-binary loop that skipped the forced non-zero exit would pass every current test.
- **MEDIUM, design.** Location: `internal/reedengine/bindings_test.go:260-266` against `bindings.go:275-283`. The `TestBinaryExecutable` row asserts only the toggle and button lines. The switch (`prev`/`next`) and follow lines, the window command and the pane prelude have no row showing they refuse a test binary. A regression of any of them to the raw lookup would not fail a test.
- **LOW, design.** Location: `internal/loomcli/looparm.go` `loopExecutable` and `internal/fronttask/hub.go` `startAction`. Their refusal and error paths are untested: loom's Warn and empty executable, and fronttask's "locate the lym executable" error. Evidence: `cli_test.go:327` and `hub_test.go:67` use stand-ins only.
- **LOW, design.** Location: `internal/proc/envmarker_linux.go:18-21, 29-31`. The `/proc`-unreadable error path and the skipped unreadable-environ path have no row.
- **NIT, design.** Location: `internal/selfexec/binary.go:34-36`. The branch where the test binary's own path does not resolve, which refuses everything, is unpinned.

Tool calls (1):
1. One `grep`/`git diff` on the test files, to check which tests reference `processCapped`, `LymBinary` and `ErrTestBinary` outside `selfexec`, and to read the test-side diffs in `loomcli`, `fronttask` and `reedengine`. My context held only the non-test side of those packages.
