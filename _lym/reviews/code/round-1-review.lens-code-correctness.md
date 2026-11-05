```yaml
lens: code-correctness
outcome: reported
```

**Lens: code-correctness.** I ran each changed function or branch with a concrete input and checked the result. I found no defects.

**Per function or branch (case ran, verdict)**

- **`Runner.SessionIdle`, recordless strand, live, tag `"second"`:** `FindRun` returns `ErrNoRunRecord`, `liveStrand` returns the strand, `state.Engine="second"`, and `engineFor` picks `second`. `findErr != nil`, so it takes the pane-fallback path. Correct.
- **`SessionIdle`, recordless strand, empty provider tag:** `state.Engine=""` resolves to the default engine. This matches `Runner.Send` and `interruptEngine`. Correct.
- **`SessionIdle`, recordless strand, unknown tag:** `engineFor` returns `ErrEngineNotRegistered`, which is returned unwrapped, as in the other callers. Correct.
- **`SessionIdle`, recordless strand, not live or untracked:** it now returns `notShuttleOrLiveStrandError`, wrapping the `liveStrand` error. `TestRunner_SessionIdle_DeadStrandErrors` only requires a non-nil error. The wording changed but is consistent with `Send`. Correct.
- **`SessionIdle`, `FindRun` fails with something other than `ErrNoRunRecord`:** unchanged. The default engine and the pane fallback still apply, and the error is not surfaced. This is an existing gap, not part of the diff.
- **`SessionIdle` calls `liveStrand` and then `requireLiveStrand`, so reed's `Status` is read twice.** If the strand dies between the two calls, the second call errors, which is the safe result. The fake's `StatusQueue` sticks on its last value, so the new test is unaffected. Correct.
- **`buildLaunchCmd` with Codex exiting non-zero:** `Chain` joins with `; `, so the exit record is appended whatever the exit status, then the shell exits. Correct.
- **`buildLaunchCmd` outside the engine path:** `Prepare` passes `eventsPath` equal to the hook's own `filepath.Join(absRunDir, "events.jsonl")`, so the two paths are identical.
- **`OpenLaunch` (`codexengine.go:162`):** it calls `codexCommand` alone, so an empty `eventsPath` is never rendered. Correct.
- **`posixShell.AppendLine`, line `100% it's` and a path containing `'`:** `printf '%s\n' '100% it'\''s' >> '/a'\''b/...'`. The `%` is never read as a format string and the quoting is sound. Correct.
- **`pwshShell.AppendLine`:** `Add-Content -LiteralPath '…' -Value '{"hook_event_name":"LymExit"}'`. The double quotes sit inside a single-quoted string, `-LiteralPath` avoids wildcard expansion, and `Add-Content` adds a newline and creates an absent file. Correct.
- **`ParseSessionSignals`, exit line with no stamp, trailing stamp lines pending before it:** the line parses, `holdFrom` resets to -1, and no stamp matches, so `At` is zero. The signal is `SessionEnd`, `Reason="codex-exit"`, `EndsProcess=true`, and `consumed` advances past the line. Correct.
- **`ReadEvents`, `stampEventTimes`:** the exit signal has `Event` false, so the marked-signal count still equals the event count. Correct.
- **`SessionFold.applySignal` and `settle`, exit record newest:** `applySignal` sets `derived` to `SessionDead` with cause `codex-exit` and a zero `Since`. `settle` hits the process-ending-session-end case first and settles that state. A Codex run therefore reads `dead` in `lym shuttle state` despite `LivenessUnproven`. Correct.
- **Wait loop, Codex exits cleanly with outputs present:** `sessionGone` is true when the exit record lies past the first-tick size and liveness is not alive. The caller (`wait.go:1411-1416`) checks `allOutputFilesExist` first and returns `OutcomeDone`, else `OutcomeDied`. That matches the completion-signal rule. Correct.
- **Wait loop, exit record at or before the first-call size:** it is not counted by `sessionGone`, which is the existing semantics for a wait that starts after the exit. The state reader still shows `dead`. Correct.
- **Wait loop, exit record, session id:** the exit record carries no session id, so `sessionGone` keeps the newest earlier non-empty id. Correct.
- **`codexCommand`, `-c 'sandbox_workspace_write.network_access=true'`:** it is a valid TOML bool and is quoted by `sh.Quote`, so it is right for both dialects. Its position before `writable_roots` does not matter. Correct.

**Tool calls**
1. `grep` of `sessionGone()` callers in `wait.go` and of `buildLaunchCmd`/`codexCommand` use: the manifest lacked the caller's output-file check and whether `OpenLaunch` can render an empty `eventsPath`.
2. `grep` of `TestRunner_SessionIdle_DeadStrandErrors` and the `fakeReed` `StatusQueue` semantics: the manifest lacked the existing dead-strand assertion and whether repeated `Status` calls are safe in the new test.
