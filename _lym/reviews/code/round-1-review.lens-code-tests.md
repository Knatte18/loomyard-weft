```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test that pins it, or none.

**Per behaviour**
1. `isBackgroundLaunch` counts an Agent or Task launch with `subagent_type: fork`.
   - Pinned by the fork row added in `TestTranscriptHasUnmatchedLaunch_BashAndAgentLaunches` (`background_test.go`, around line 169).
   - It would fail if the fork clause were dropped.
2. `Claude.OutstandingForks` and `unfinishedForks`.
   - Pinned by `TestOutstandingForks` (`background_test.go`).
   - Rows: fork with no notification, fork named by a queued notification, shell excluded, missing transcript is an error.
   - A fork launch whose tool result is not yet written is not covered. That case is where the wait could answer ready too early.
3. The `awaitForks` poll loop.
   - Pinned by `TestAwaitForks` (`await_test.go`).
   - Rows: ready at once, ready after N reads, cap elapsed with the last sleep trimmed to the remaining cap, read error, cancelled context.
   - It uses the told `awaitClock`, the same fake clock the `TestAwaitAny` file-poll tests use.
4. `Runner.AwaitForks` error paths and wiring.
   - Pinned by: none. Uncovered paths:
     - `toldErr`
     - unknown guid (`FindRun` error)
     - engine without `OutstandingForkReader`
     - passing `state.SessionID` and `paneCwd` to the reader
5. The `lym shuttle await-forks` verb (`shuttlecli/awaitforks.go`).
   - Pinned by: none. Uncovered behaviours:
     - registration in `cli.go`
     - the no-strand-env error
     - the resolve-failure error
     - the engine-error envelope
     - the `ready: false` envelope with `outstanding`
     - the `ready: true` envelope
     - the `--cap` default
6. `c.reed` population in `PersistentPreRunE`.
   - Pinned by: none directly. It is only reachable through the verb, which is untested.
7. The `taskFields` extraction from `stateFields` in `state.go`.
   - My grep found no `shuttlecli` test that mentions `taskFields` or `stateFields`. It did not find a test of the `state` verb's `outstanding` output either.
   - This is a pure refactor with the same output. I found no test that would fail if its shape changed.
8. `isBackgroundLaunch` now changes `Event.Outstanding` for a Stop payload with no `background_tasks` list. That feeds `events.go:49` and `signals.go:154`, and from there the session-state busy/background decision.
   - Pinned by: none at that level. The `smoke_turns_test.go` fork test (`TestSmokeTurnsFork`) uses the payload signal. No test feeds a Stop payload with no list and a transcript holding a fork launch without `run_in_background`.
9. Stencil and doc changes are static prose, so they are exempt.

**Findings**
- **F-T1: MEDIUM, design.** Location: `internal/shuttlecli/awaitforks.go:110-164`. The new `await-forks` verb has no test. The existing sibling `TestAwaitVerb` in `cli_test.go` (around line 259) covers only the file `await`. The no-strand error, the resolve error, both envelopes and the registration are unpinned. Evidence: the grep over `internal/shuttlecli/*_test.go` shows no `await-forks` or `AwaitForks` reference, and the `AwaitForks`/`OutstandingFork` test grep lists only `background_test.go` and `await_test.go`.
- **F-T2: MEDIUM, design.** Location: `internal/shuttleengine/await.go:266-285`. `Runner.AwaitForks` has no test. Its unknown-guid error, its missing-capability error and its `toldErr` return are unpinned. Only the inner `awaitForks` is tested.
- **F-T3: LOW, design.** Location: `internal/shuttleengine/claudeengine/background.go:267`. The behaviour change for sessions that do not call await-forks (a fork launch counted from the transcript fallback of a Stop payload with no list) is pinned only at `transcriptHasUnmatchedLaunch`. Add a row to the events or `signals` tests that parse a Stop payload with no `background_tasks` list over a transcript holding a bare fork launch. That row should show the turn end reads waiting.
- **F-T4: LOW, design.** Location: `internal/shuttleengine/claudeengine/background_test.go` (`TestOutstandingForks`). Add a row for a fork launch whose tool result is not yet written. It decides whether `OutstandingForks` answers empty (wait returns ready early) or outstanding.

**Tool calls**
1. Grep of `internal/shuttlecli/*_test.go` for `outstanding`, `taskFields` and `stateFields`, plus a grep listing test files that mention `OutstandingFork` or `AwaitForks`. My context did not show the shuttlecli tests or any other test referencing the new symbols.
