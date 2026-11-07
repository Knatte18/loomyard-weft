```yaml
lens: fix-code-correctness
outcome: reported
```

Task: run the correctness lens over the fixer's change to `batchingnudge_test.go`, `batchingnudge.go` and `doc.go`.

I traced each new test row, the changed sleep closure and the new constructor test step by step against the code in context. I also read `latestReading` in `usage.go`, which my context lacked.

**Traces, no findings:**
- **`NoStateFileIsSilentWithoutWaitingForALateCall`:** it passes with the order as it stands. If the wait ran before the state read, the first tail would pass the threshold check and sleep 1 would write the call, so `wantSleeps` 0 would fail.
- **`SubagentPastItsOwnCapIsSilentWithoutWaitingForALateCall`:** it passes with the order as it stands. A reorder would fail it the same way.
- **`SubagentEditReachingTheTranscriptDuringTheWaitFires`:**
  - At attempt 0 the agent transcript holds the Edit message as a sidechain line at 200000 tokens.
  - The user lines carry no `isSidechain` and are skipped.
  - Sleep 1 writes the full transcript, the Write message fires, and the count lands under the agent's id.
- **`SubagentWithNoReadingYetIsSilentWithoutWaiting`:**
  - The agent file holds only `"\n"`, and `latestReading` skips the blank line and returns not known.
  - The function returns false with no sleep.
  - Removing the first-tail check would make it wait 3 times, so the row fails with the defect in place.
- **`EditReachingTheTranscriptAfterSeveralRereadsFires`:**
  - Sleep 1 writes the written messages plus one sidechain message. The size changes, the file is reread, and the main-chain tail does not hold the call.
  - Sleep 2 writes the full transcript, the next pass finds the call, and `wantSleeps` is 2.
  - A gate that never rereads would fail it. The extra lines are sidechain lines, so `tailMessages` skips them in a main-chain transcript.
- **`TestEngineConstructorsSetTheTranscriptWait`:** it fails if either constructor drops `transcriptWait`, because the zero wait has tick 0, attempts 0 and a nil sleep.
- **Sleep closure:** `max(tt.lateAfter, 1)` and `for range sleeps` need Go 1.21 and 1.22. The package already relies on 1.22 loop-variable semantics.
- **Callers:** no signature changed, so no caller is affected.

**Finding:**

- **LOW** — `doc.go` (the added sentence after "It also waits only while the tail's first reading…") and the `awaitCallInTranscript` doc comment in `batchingnudge.go`.
  - **Summary:** both sentences say the first-tail reading "is the previous message's", with no condition.
  - **Evidence:** the reading comes from `latestReading` on the first tail. When the hook runs late and the call's own message is already in that tail, the reading is the call's own message's, and the call is not silenced. The review's own text says "usually". The sentences state the common case as the rule.
  - **Fix:** change to "That reading is usually the previous message's, so …".

Tool calls made:
1. `grep -n "func latestReading" -A40` on `internal/shuttleengine/claudeengine/*.go`: my context lacked `latestReading`, which decides the empty-transcript row and the sidechain filtering.
