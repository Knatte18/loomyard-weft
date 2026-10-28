```yaml
lens: fix-code-correctness
outcome: reported
```

Lens fix-code-correctness: I traced each changed branch with a concrete input and found no wrong results. One LOW remains: a test that does not pin its behavior.

**F1 (`WaiterGone`)**
- `ReadAgentActivity` sets `WaiterGone = true` in the `!runIsLive` branch. A running record with a dead pid, or with no pid, reads `WaiterGone` with no API error and no usage limit.
- `agentActivityFor` skips such a reading, so `live` stays false and `newest` stays zero. `liveRunStrands` skips it too.
- `battencli/wire.go:605` ignores the field, so the idle notice still reads the run.
- The changed rows in `activity_test.go` (lines 236, 247, 248) fail without the `WaiterGone = true` line, and the live rows keep the zero value.
- The `WaiterGone` skip in `agentActivityFor` and `liveRunStrands` has no test row, so reverting it would pass the suite. The review's fix asked only for the `ReadAgentActivity` row, so this is not a regression.

**F3 (`typedInstruction`)**
- `/compact` gives a first word with no further `/`, so it is a command and returns "". A lone `/` is also a command.
- `/tmp/x.go fails to build` has a second `/` in its first word, so it is returned as the instruction.
- The new `signals_test.go` row fails with the old `HasPrefix("/")` check.

**F4 (`operatorInstructions`)**
- The deferred prompt line is sent through `typeVerified` (`run.go:1605`), which records it in `sent-texts` before typing. A delivered prompt is therefore excluded twice, once by the prompt match and once by the sent filter. A mismatch between the hook's text and `PromptLine` would still not carry the prompt.
- With `PromptLine` empty, the first turn start is still skipped as before.
- With `PromptLine` set and never delivered, `fix the typo` does not match, so it is carried.
- The "died before its deferred prompt landed" row fails with the old unconditional skip. The "delivered" row also passes with the old code, so it is only a guard.

**F2/F5 (carry-over tests)**
- The newest-run rows fail if the code picks the oldest run or any unfinished run. The died and timeout rows fail if `unfinished` drops those outcomes.
- The start-retry test builds the prompt once before the loop, so both `PrepareCalls` carry it. It expects one notice after success and one Warn line.
- That Warn count would fail if `noticeCarriedInstructions` ran per attempt or logged twice.

**LOW — the "a run directory that cannot be written does not block the send" row in `TestRun_Send_RecordsTheSentText`**
- `recordSentText` returns nothing and only logs, so no code path lets its failure fail `Send`. The row passes whether or not the append failure is tolerated.
- The `os.IsNotExist` check on the file does show that no file was created. The row cannot catch a future change that propagates the error unless it also asserts the Warn log line.

Further tool calls:
1. `grep PromptLine` in `internal/shuttleengine/*.go`, plus `run.go:1595-1610`. I read these because my context did not show how the prompt line is sent or whether it is recorded in `sent-texts`.
