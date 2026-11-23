```yaml
lens: code-tests
outcome: reported
```

Task: review the diff's tests (code-tests lens). For each behaviour the diff adds or changes, I name the test row that would fail if it broke.

**Behaviours and the tests that pin them:**

1. **`isTopBoxRule` takes a top rule shaped `─ label ─` with one rule glyph on each side.** Pinned by the `TestInputBoxText` row "a label filling the top rule but for one glyph on each side" (`internal/shuttleengine/claudeengine/session_test.go:221-228`). I traced it on the old code: the line has no `───` prefix, no `… ─` suffix, and fails `isBoxRule`. Nothing above the caret counts as a top rule, so `ok=false` and the row fails as it should.
2. **A line not ending in a rule glyph is still no top rule.** Pinned by the existing row "a cut label with no rule glyph after it is no top rule" (`session_test.go:229`).
3. **The notice goes out when the retry cap runs out.** Pinned by the `TestStartup_SendFailuresTearDown` row "a prompt that stays busy is sent again up to the retry cap" with `wantHeldNotice: true` (`internal/shuttleengine/startup_test.go:2276`), asserted at `startup_test.go:2375-2387`.
4. **The notice goes out when the run's deadline cuts the retries short.** Pinned by the row "a busy prompt send past the run's deadline is not sent again" with `wantHeldNotice: true` (`startup_test.go:2277`).
5. **Exactly one notice is sent, and it names the strand.** Pinned by `len(notices) != wantNotices` and the substring "agent strand-1 never took its start prompt" (`startup_test.go:2375-2387`).
6. **No notice when a busy hold clears on a retry and the prompt lands.** Pinned by the row "a prompt held busy for one send window is sent again" (`startup_test.go:2275`), which expects zero notices.
7. **No notice for an error that is not a `*SessionBusyError`.** This covers the not-landed, foreign-box, persist-failure and skill-load rows. Pinned by those rows expecting zero notices (`startup_test.go:2268-2273`, `2279`).
8. **No notice for an interactive run or a runner with no notifier.** This is the guard at `run.go:957`. Pinned by: none.
9. **The hold's reason in the notice is wrapped by `delimitedAgentText`.** This is the build at `run.go:964-965`. Pinned by: none. The assertion checks only the strand-name substring.
10. **The notice falls back to the strand GUID when `StrandName` is empty** (`run.go:960-963`). Pinned by: none. This fallback only applies to an old record, and a fresh start always records a strand name, so I file no finding for it.
11. **A notifier error is only logged** (`run.go:966-968`). Pinned by: none. The surviving mutant at `run.go:966` changes only whether a log line is written, which the rubric exempts, so no finding.

**Findings:**

- **LOW · design · `internal/shuttleengine/run.go:957`**
  - Summary: the contract "notifies only an autonomous run on a runner with a notifier" (doc comment at `run.go:954`) has no test. No row of `TestStartup_SendFailuresTearDown` runs an interactive spec down the held-busy path and expects zero notices.
  - Evidence: the reach report lists only `TestStartup_SendFailuresTearDown` for `noticeStartPromptHeld`, and that test always sets a notifier (`startup_test.go:2340-2344`). The mutation report counts 2 uncovered changed blocks.
  - Fix: add an `interactive` field and one interactive row that stays held busy and expects no notice.
- **LOW · design · `internal/shuttleengine/run.go:964-965`**
  - Summary: no test checks that the notice carries the last hold's reason inside the agent-text delimiters. The doc comment (`run.go:953`) gives this as the reason for delimiting, since the reason can quote the input box. If the `delimitedAgentText` call were dropped, or the raw reason passed instead, every row would still pass.
  - Evidence: the assertion at `startup_test.go:2377` matches only "agent strand-1 never took its start prompt".
  - Fix: on the retry-cap row, also assert that the notice contains `«` + the hold's reason + `»`.

Tool calls: none. Everything came from the inherited context.
