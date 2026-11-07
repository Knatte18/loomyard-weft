```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test row that pins it, or say none.

**Per behaviour, the test that pins it**

| Behaviour | Pinning test |
|---|---|
| A call that reaches the transcript during the wait fires | `TestBatchingNudge/EditReachingTheTranscriptDuringTheWaitFires`, `wantSleeps: 1`. |
| The wait stops at the attempt bound and stays silent | `ToolUseIDAbsentFromTheTailIsSilentAfterTheBoundedWait`, `wantSleeps: nudgeWaitAttempts`. |
| No wait below `context_tokens`, judged on the first tail | `LateCallBelowTheContextThresholdIsSilentWithoutWaiting`, main session only. |
| No wait once the chain is past its cap, read before the wait | `LateCallPastTheNudgeCapIsSilentWithoutWaiting`, main session only. |
| No wait and no nudge for a run with no state file | `NoStateFileIsSilent`, but it does not pin the ordering (finding 1). |
| An unreadable transcript is silent at once (the stat failure) | `UnreadableTranscriptIsSilentd`, which expects 0 sleeps by the default `wantSleeps`. |
| A corrupt state file is an error, now raised by `ReadJSON` | `CorruptStateFileIsAnError` and `BashCallThatIsNoLookupLeavesTheStateUnread`. `TestHookBatchingVerb/CorruptStateIsReportedAndExitsZero` pins it at the verb. |
| The size-change gate on rereads | None. No row is observable through the gate (finding 3). |
| `New` and `NewFromConfig` set `transcriptWait` | None (finding 2). |
| The zero `Claude` value never waits | None. It is safe by construction, because `attempts` is 0 and `sleep` is never called. |
| The `current < 0` guard dropped inside `UpdateJSON` | None, and none is needed. It rests on `awaitCallInTranscript` returning only a tail that holds the call. |
| A subagent (sidechain) late call: the first-tail reading and message lookup on the agent's transcript | None. No row sets both `late` and `agentID`. |

**Findings**

1. **LOW, design.** The no-state-before-wait ordering is unpinned. The row is `NoStateFileIsSilent` at `batchingnudge_test.go:286`. Its fixture holds the call in the transcript, so `awaitCallInTranscript` would return at attempt 0 with 0 sleeps even if the state read came after the wait. A row with `noState: true` and `late: true` and `wantSleeps` of 0 would fail on a reorder. The same applies to the subagent cap: `seeded.count(input.AgentID)` has no late subagent row. The evidence is `batchingnudge.go:255-262` against the rows at `:277` and `:303`.

2. **LOW, design.** Production engines waiting is unpinned. `TestBatchingNudge` builds `New()` and then overwrites `engine.transcriptWait` (`batchingnudge_test.go:367-368`). If `New` or `NewFromConfig` lost the `transcriptWait: defaultBatchingTranscriptWait()` line, no test would fail, and the hook would silently return to the original bug. The fix is an assertion that both constructors return a non-zero `transcriptWait`, as a row or assertion in an existing test of the constructors. Evidence: `claudeengine.go:54-68`.

3. **LOW, design.** The wait loop's multi-attempt paths are unpinned. The late row writes the whole transcript at the first sleep, so only `attempt` 1 is exercised. No row has the file grow without the call and then hold it at a later sleep, which would pin continued polling after a grown-but-still-missing read. No row pins the first-tail case where the chain has no usage reading yet (`!reading.Known`), which returns false with no wait. Evidence: `batchingnudge.go:313-327` and the late fixture at `batchingnudge_test.go:366-373`.

4. **NIT, design.** `hookbatching_test.go:83` now expects `"the batching nudge state"`. That substring is shared by the old and new error wrappers, so it still pins the error path but would no longer distinguish them. It is not a coverage gap, and the corrupt-state error text is also asserted nowhere else.

**Test economy**
- The two new `late` rows cover behaviour no existing row covers: no wait. They are not duplicates of `BelowTheContextThresholdIsSilent` or `NudgeCapReachedIsSilent`, since the transcript there holds the call and nothing waits.
- `wantSleeps` of 0 on rows whose call is already in the transcript is trivially true. Only the late rows and the absent-call row make that assertion meaningful, and it is not a duplicate of another assertion.

**Tool calls:** none. The manifest context held everything I needed.
