```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, the test that pins it, or none.

**Per behaviour**

1. **`SessionIdle` reads a recordless strand through its live provider's engine.**
   - Pinned by `TestRunner_SessionIdle_RecordlessStrandReadsThroughProvider` (`session_test.go:231-254`).
   - It would fail if the code went back to the default engine, because only the "second" engine reports idle and the default engine's captures must stay empty.

2. **`SessionIdle` on a recordless strand reed does not track as live fails with `notShuttleOrLiveStrandError`.**
   - Pinned by `TestRunner_SessionIdle_DeadStrandErrors` (`session_test.go:~223`) only if that fixture is recordless. `withStrand` may or may not write a run record, and I could not tell.
   - That test checks `err == nil`, not the error's wording.
   - The rejection of an untracked guid has no named test of its own. This is LOW, and it is a gap only if `withStrand` writes a record.

3. **`SessionIdle` on a recordless strand whose provider tag names an unregistered engine returns `engineFor`'s lookup error.**
   - Pinned by none. This is the focus directive's "unknown tag" case.
   - Severity LOW, class `scope`.
   - Fix: add a row or assertion in the recordless-strand test with `Provider = "unknown"`, expecting an error wrapping `ErrEngineNotRegistered`, and expecting no capture to be read.

4. **`SessionIdle` on a recordless strand with an empty provider tag resolves to the default engine.**
   - Pinned by none for the new branch. The new test sets "second" only.
   - If the empty-tag case is covered by pre-existing recordless tests, this is covered.

5. **The exit record parses as a process-ending session end.**
   - Pinned by `TestParseSessionSignals`, with `exitRecordLine` appended and the `SessionSignalSessionEnd` / `exitRecordReason` / `EndsProcess` want row (`events_test.go:65, 84`).
   - The test's file-level doc already says the row has no time.

6. **A Codex run whose loop ended reads `dead` in `lym shuttle state`.**
   - Pinned by none end to end. Nothing feeds a Codex events file ending in the exit record through `readRunSessionState` or `SessionFold` to assert `SessionDead` with cause `codex-exit`, and nothing does so through `Run.sessionGone` in the wait loop.
   - The fold's `EndsProcess` handling (`sessionstate.go:187-196, 286`) is tested elsewhere, but only with Claude's signal. So a Codex-specific break in how `ParseSessionSignals` output reaches the fold would go uncaught.
   - Severity MEDIUM, class `scope`. This is Goal fault 2's observable behaviour.
   - Fix: add one row to an existing session-state test in `codexengine`, or to `readRunSessionState`'s tests with the Codex engine. The events file would hold a SessionStart, a Stop and the exit record, and the row would expect `Dead` with cause `codex-exit`.

7. **The launch line appends the exit record between Codex and `exit`.**
   - Pinned by `TestBuildLaunchCmd`, with literal want strings on two rows (`command_test.go:26-33`). It would fail if the append, its order or its quoting changed.
   - `TestPrepare_Launch` and `TestPrepare_InstallsGuardsFromItsInputs` rebuild the expected line through `buildLaunchCmd`, which Prepare itself calls. They therefore do not check Prepare's own `eventsPath` wiring against a literal.
   - The wiring is `filepath.Join(absRunDir, "events.jsonl")` at `codexengine.go:125`, and it is the same path the hooks use. A wrong `eventsPath` would fail only if it differed from the test's own expectation, and the test uses the same path.
   - This is acceptable, so I raise no finding.

8. **`skillload_test`'s assertion weakened from `HasSuffix(..., "'; exit")` to `Contains(..., "'; ")`.**
   - The test still pins the load line as the first turn and the absence of the pointer.
   - It no longer pins that the line ends in `exit`. `TestBuildLaunchCmd` pins that, so there is no coverage hole.
   - NIT only, and I raise no finding.

9. **The network setting is on the launch line.**
   - Pinned by `TestBuildLaunchCmd`'s two launch rows and by its opened-session row (`command_test.go:47`).
   - The opened row tests `codexCommand` directly.
   - `open_test.go:85` compares `OpenLaunch`'s whole `Launch`, and I assume it builds its expected command through `codexCommand`. That would not check the opened line's absence of an exit record against a literal.
   - `TestBuildLaunchCmd`'s opened row has no `eventsPath` and no exit, so it pins the absence of both.
   - Covered well enough, so I raise no finding.

10. **`Shell.AppendLine`, POSIX and pwsh.**
    - Pinned by `TestShell_AppendLine` (`shell_test.go:120-143`), which has four rows: both dialects' plain case, pwsh quote doubling, and POSIX `%` plus quote.
    - Nothing pins that it creates an absent file or ends the line. That is shell semantics, not testable as a string.
    - No gap.

11. **Other `Shell` implementers keep working.**
    - Only `posixShell` and `pwshShell` implement the interface, and both gained the method. The compiler enforces this.
    - No finding.

**Findings**

- **MEDIUM, scope.** `internal/shuttleengine/codexengine` (no test exists): no test takes a Codex events file ending in the exit record through the session-state fold or `sessionGone` to `dead`. The only evidence is `events_test.go:65-84`, which stops at the parser. See item 6 for the fix.
- **LOW, scope.** `internal/shuttleengine/session_test.go:231-254`: no row for a recordless strand whose provider tag is unregistered, which is `SessionIdle`'s `engineFor` failure path. The fix is in item 3.
- **LOW, scope.** `internal/shuttleengine/session_test.go:~223`: the new `errors.Is(findErr, ErrNoRunRecord)` branch's live-strand failure (`notShuttleOrLiveStrandError`) is covered only if `withStrand` is recordless, and the test asserts `err != nil` only. If `withStrand` is not recordless, add a row with an untracked guid and no record, expecting an error that contains "not a shuttle strand and not a live reed strand".

**Tool calls**
1. A grep of `OpenLaunch` in the codexengine tests plus a read of `session_test.go:195-230`. My context lacked the open test and the existing `SessionIdle` dead-strand test.
