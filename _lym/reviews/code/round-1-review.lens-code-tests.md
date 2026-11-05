```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I looked for the test row that would fail if it broke.

**Per behaviour**
- **`readinessState` falls back to the whole-file fold when no signal lies past a non-zero offset.** Pinned by two rows, both for the start path:
  - `TestStartup_SkillLoadTurns`, row "the hook state's idle reading after the load turn sends the prompt into a pane the pane fallback refuses" (`startup_test.go:1597`). It would fail with `ErrSessionBusy` if the fallback were removed, because `IdleSession` no longer returns true for `LOAD:` captures.
  - The hook-state row of `TestRunner_Start_TypesTheSegmentColorAfterStartup`. I did not open this row; I relied on the summary's description of it.
- **Offset is zero, so there is no fallback.** None. If the `rs.PromptOffset == 0` guard were dropped, the fallback would be a repeat read of the same file and no test would fail. This is a no-op branch, so I rate it LOW.
- **Cause is not `no-signal`, so there is no fallback.** None. Examples are the `unsupported` cause for an engine without a parser, and an unreadable events file. If the cause guard were dropped, no test would fail, and an `unsupported` or events-unreadable state would be re-read for nothing, but it would stay unknown.
- **No signal past the offset and none before it, so the state stays unknown with cause `no-signal`.** None. This is the whole-file fold still reading unknown, which then goes to the pane fallback. `sessionstate_test.go:75` pins `SessionCauseNoSignal` only for the fold itself, not through `readinessState`.
- **A stale turn start before the offset, left unmatched, or a stale turn end.** None. This is the staleness risk the focus file raises. No test puts an unmatched turn start before the offset and checks that the send is held or released correctly. The start-path row has only a clean load turn end before the offset.
- **`Runner.SessionIdle` reads the same state as the send path.** None. It goes through `sessionReadiness` (`session.go:266`), and no test in the diff drives `SessionIdle` with a non-zero offset and no later signal.
- **Fake change: `hookStateSkillEngine.IdleSession` no longer counts `LOAD:` as idle.**
  - It tightens the fake, since the pane fallback now refuses throughout. The two hook-state rows therefore go through the hook fold for every send, and that is what pins the fix.
  - I found no weakened assertion on the pane fallback. Only hook-state rows use this engine.
  - Because the pane never reads idle, those rows no longer show that the color send and the retry send work before the offset moves. They are still covered by the fold of the launch-carried turn's end, per the new fake comment.
- **`doc.go` sentence.** Static prose, so exempt.

**Findings**
1. **MEDIUM, scope.** `internal/shuttleengine/readiness.go:129-136`. `readinessState` has no direct test for its branches: zero offset, a cause other than `no-signal`, and a still-unknown whole-file fold. Only the happy-path fallback is covered, and only indirectly through the start fixture. Add rows to a table-driven test of `sessionReadiness` or `Runner.SessionIdle`. They should cover: a non-zero offset with no later signal and a load turn end before it, which reads ready; a zero offset with an empty file, which reads unknown; and an unparsing engine, which reads unknown with cause `unsupported`. Evidence: the rows at `startup_test.go:1597` and `sessionstate_test.go:75`.
2. **MEDIUM, design.** `internal/shuttleengine/readiness.go:131-135`. No test pins the stale-state risk. For example, no test has an unmatched turn start before the offset with no signal after it. The fallback would then fold that stale start, and `sessionReadiness` would hold the send through `unmatchedTurnStart`. A row should state the intended result, whether held or released. Evidence: the only offset-bearing fixture is the clean load turn in `skillStartFixtureWith`, `startup_test.go:1449-1457`.

**Tool calls**
1. Read `startup_test.go:1430-1630`, because the hook-state test rows were not in my context.
2. Grep for `SessionCauseNoSignal` in the shuttleengine tests, to find existing coverage of the `no-signal` cause.
