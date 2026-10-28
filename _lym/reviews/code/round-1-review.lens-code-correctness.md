```yaml
lens: code-correctness
outcome: reported
```

Task: judge the correctness of commit b1fe0b2d, the test-only serial sink test in `internal/fabriccli/hubmutation_integration_test.go`.

**Lens list (function or branch, case run, verdict):**

- **`TestRunCLI_BypassPushPointsTheTraceSinkAtItsHub`, race with the parallel tests:** I ran it against `TestRunCLI_Anchor` and `TestRunCLI_HubMutationScenario`, both top-level `t.Parallel()` tests. Go pauses a parallel top-level test at its `t.Parallel()` call until every serial top-level test has finished. The new serial test therefore finishes its push and its `logger.TraceDir()` read before any parallel test body resumes. Correct. The old read raced because it sat inside the parallel scenario.
- **Same test, other serial tests:** I checked tests that also call `SetDurableSinkDir`, such as `anchor_integration_test.go:33` and `weft_verbs.go:265`. Serial tests run one after another, and each restores the sink in `t.Cleanup`. Correct.
- **Same test, sink restore:** `t.Cleanup(func() { logger.SetDurableSinkDir("") })` is registered after `hubforge.CopyHub`. Cleanups run last-in-first-out, so the sink is cleared before the hub directories are removed. `weft_verbs.go:265` is the only global the bypass branch repoints. Correct.
- **Same test, push with nothing unpushed:** I ran the push with no extra commit on the prime. `CopyHub` returns a hub with the prime already pushed, so the bypass push has nothing to send. `weft_verbs.go:265` sets the sink before `DetachedPush`, so the sink assertion does not depend on how that call goes. Correct. The test still asserts exit 0 and `envelope.RequireOK`, so a push failure shows up as a test failure.
- **Same test, expected value:** `fabricengine.HubLogsDir(h.Path)` matches the `HubLogsDir(hubLocation.HubPath)` that the verb sets. This assumes `h.Path` is the hub path the verb resolves from `h.BoardDir()`. The old assertion in the scenario used the same expression, so this is unchanged and correct.
- **Scenario step `BypassPushAdvancesBothUpstreams`, after the sink check was removed:** The weft and warp bare-repo SHA assertions, the exit-code check and `envelope.RequireOK` all remain. The step still has its own `t.Cleanup` for the sink. The edited comment now matches the code. Correct.

No findings.

**Tool calls (1):**
1. `Bash`: read `weft_verbs.go:245-280`, `anchor_integration_test.go:20-60` and `logger/sink.go:225-240`, for the bypass branch's sink write, the parallel anchor tests and `TraceDir`, none of which my context held.
