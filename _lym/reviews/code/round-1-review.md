---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/claudeengine/claudeengine.go:59
    summary: "No test pins that New and NewFromConfig set transcriptWait, so dropping that line silently restores the original live gap"
    fix: >-
      In internal/shuttleengine/claudeengine/batchingnudge_test.go add one table-driven test,
      TestEngineConstructorsSetTheTranscriptWait, opening with tiergate.Require(t, 0) and t.Parallel().
      Its rows are New() and NewFromConfig(shuttleengine.Config{}, configengine.Dirs{}), both already imported in that file.
      For each engine assert transcriptWait.tick == batchingTranscriptTick,
      transcriptWait.attempts == batchingTranscriptAttempts and transcriptWait.sleep != nil.
      Do not drive BatchingNudge for this, since an absent call would wait out the production bound.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/claudeengine/batchingnudge_test.go:277
    summary: "The state-read-before-wait order and the first-tail checks on a sidechain or still-empty transcript are unpinned"
    fix: >-
      In TestBatchingNudge in internal/shuttleengine/claudeengine/batchingnudge_test.go add rows after
      LateCallPastTheNudgeCapIsSilentWithoutWaiting, each with wantSleeps left at 0 unless stated.
      Row NoStateFileIsSilentWithoutWaitingForALateCall, with tokens 200000, messages lookupChain, late true and noState true.
      Row SubagentPastItsOwnCapIsSilentWithoutWaitingForALateCall, with tokens 200000, messages lookupChain,
      late true, agentID nudgeAgentID and before set to withCount(nudgeAgentID, batchingNudgeCount{Nudges: 2}).
      Row SubagentEditReachingTheTranscriptDuringTheWaitFires, with tokens 200000, messages of an Edit message then a Write message,
      late true, agentID nudgeAgentID, wantSleeps 1 and want the editChainNudge line for 2.
      Row SubagentWithNoReadingYetIsSilentWithoutWaiting, with tokens 200000, one Grep message,
      late true and agentID nudgeAgentID, so the agent transcript holds no usage line at the first tail.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/claudeengine/batchingnudge_test.go:368
    summary: "No row has the transcript grow without the call before the call arrives, so polling past the first reread is untested"
    fix: >-
      In TestBatchingNudge add a table field lateAfter, the sleep count at which the late message is written, with zero meaning 1.
      In the injected sleep function, for each sleep before lateAfter rewrite the chain transcript
      with the messages written so far plus one extra nudgeSidechain message, so the file grows without holding the call,
      and at sleep number lateAfter write the full tt.messages.
      Add the row EditReachingTheTranscriptAfterSeveralRereadsFires, with the edit chain of
      EditReachingTheTranscriptDuringTheWaitFires, late true, lateAfter 2 and wantSleeps 2.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/claudeengine/batchingnudge.go:319
    summary: "The first-tail context check reads the previous message's usage, so the one call whose message first reaches context_tokens is silenced without a wait"
    fix: >-
      Keep the check, since it spares every chain below the threshold the wait, and state its consequence.
      In internal/shuttleengine/claudeengine/doc.go, after the sentence that ends
      "It also waits only while the tail's first reading is at or above `context_tokens`.",
      add the sentence "That reading is the previous message's, so the one call whose message first reaches `context_tokens` is left silent."
      In the doc comment of awaitCallInTranscript in batchingnudge.go, add the same fact after the line
      "It also reports false at once for a chain whose context is below contextTokens by the first tail's latest reading."
  - id: F5
    severity: LOW
    class: consistency
    origin: handler
    location: internal/shuttleengine/claudeengine/batchingnudge.go:113
    summary: "The comment above the wait constants restates the tick times attempts product as two seconds in all, a derived tally that goes stale with either constant"
    fix: >-
      In internal/shuttleengine/claudeengine/batchingnudge.go replace the first two comment lines above the
      batchingTranscriptTick and batchingTranscriptAttempts constants, which read
      "The production wait for a call to reach the transcript is two seconds in all." and "That covers the write delay after a message ends.",
      with the one line "The production tick and attempts bound the wait for a call to reach the transcript, and cover the write delay after a message ends."
      Keep the third line, "It leaves out only the early calls of a message still being written."
---

### [MEDIUM] No test pins the production wait on the constructors

**Location:** internal/shuttleengine/claudeengine/claudeengine.go:59 and :65-67; internal/shuttleengine/claudeengine/batchingnudge_test.go:367-368
**Issue:** `TestBatchingNudge` builds `New()` and then overwrites `engine.transcriptWait` with its own fast wait, and `TestHookBatchingVerb` finds every call at attempt 0.
No test reads the wait the production constructors set.
If `New` or `NewFromConfig` lost `transcriptWait: defaultBatchingTranscriptWait()`, the zero value would never wait, every test would pass, and the hook would return to the live gap this change closes.
The hook reaches the engine through `NewFromConfig`, via `shuttleengines.BatchingNudger`.

### [LOW] The state-read-before-wait order and the first-tail checks on a sidechain or still-empty transcript are unpinned

**Location:** internal/shuttleengine/claudeengine/batchingnudge_test.go:277, :286, :303; internal/shuttleengine/claudeengine/batchingnudge.go:255-262
**Issue:** The summary's claim that a run with no state file never waits has no row that fails on a reorder.
`NoStateFileIsSilent` holds the call in the transcript, so it returns at attempt 0 with no sleep whichever way the state read and the wait are ordered.
Only the main session has a late row for the cap and for the threshold.
No row sets both `late` and `agentID`, so the first-tail check on a sidechain transcript, which the focus asked about, is not exercised, and neither is the first tail that holds no usage reading.

### [LOW] Polling past the first reread is untested

**Location:** internal/shuttleengine/claudeengine/batchingnudge_test.go:366-373; internal/shuttleengine/claudeengine/batchingnudge.go:313-327
**Issue:** The late row writes the whole transcript at the first sleep, so the loop runs only to its second pass.
No row grows the file without the call and then holds it at a later sleep, which is the shape the size gate and the attempt counter exist for.
The size gate is observable through that row: a gate that never reread would fail it.

### [LOW] The first-tail context check silences the call whose message first crosses the threshold

**Location:** internal/shuttleengine/claudeengine/batchingnudge.go:319-322
**Issue:** At attempt 0 the call's own message is usually not in the tail, so `latestReading` returns the previous message's usage, which is lower than the usage of the message holding the call.
A chain whose context first reaches `context_tokens` in the call's own message gets no wait and no judgment for that one call.
The closure's re-check on the final tail can only silence, never rescue, so the two readings disagree in one direction only, and by one call per chain.
The cost is small and the saving real, since every chain below the threshold skips the wait, but neither doc.go nor the function doc says so.
For a subagent the first message has no sidechain reading yet, so it is silent too; that is harmless, because a chain needs two messages to breach.

### [LOW] A derived tally in a comment

**Location:** internal/shuttleengine/claudeengine/batchingnudge.go:113
**Issue:** "two seconds in all" is the product of `batchingTranscriptTick` and `batchingTranscriptAttempts` written out as a number.
Changing either constant makes the sentence false with nothing to catch it.
doc.go already names both constants without the figure, which is the form the repository's comment standard asks for.

## Focus verdicts

- **Threshold on the first tail and again on the final tail.**
  The two readings can disagree only by silencing: the first tail holds the previous message's reading, and the final tail's reading is the call's own.
  Nothing wrongly waits, because the first check precedes every sleep.
  For a sidechain, attempt 0 reads the agent's own transcript and its sidechain lines, so a subagent with an earlier message behaves as the main session does; one with no earlier reading is silent without waiting.
  Recorded as F4 for the single-call silencing and as F2 for the missing sidechain row.
- **State read outside the lock, wait outside the lock.**
  Parallel calls of one message still nudge at most once: the closure re-reads the count and `NudgedMessage` under the write lock, so the second call sees the first's message id and returns `errNoNudge`.
  Dropping `current < 0` inside the closure is safe, because the tail the closure reads is the tail `awaitCallInTranscript` returned only after `messageHolding` found the call, and `tailMessages` is deterministic on the same bytes.
  Not a defect.
- **`awaitCallInTranscript` loop.**
  Bounded: `attempts` sleeps and `attempts + 1` reads at most.
  A truncated or rewritten file is reread because the gate is `!=`, not `>`.
  A rewrite at the same size is not reread, which an append-only transcript cannot produce.
  A stat or read failure mid-wait returns false at once, and a zero `transcriptWait` returns after one read without calling the nil `sleep`.
  Not a defect; the multi-pass shape is untested, filed as F3.
- **Goal coverage.**
  The cause the diff names, the call's message not yet in the transcript when a fast call's hook runs, explains why edit follows rarely fired live while a replay of the finished transcript did.
  The fix covers a single-edit message, whose line lands soon after the call starts, and the later edits of a parallel edit message, which see the whole message.
  The early calls of a parallel message that outlast the two-second bound stay silent, as the summary and doc.go say, and the edit-chain judgment does not need them.
  The diff changes the state read, the wait, the error text of a corrupt state file, the constructors, the tests and doc.go, and nothing beyond that.
- **Summary claims.**
  No wait for a run with no state file, a chain past its cap or a chain below `context_tokens`: borne out by the order of the reads in `BatchingNudge` and `awaitCallInTranscript`.
  `New` and `NewFromConfig` both set `transcriptWait`: borne out.
  No other production constructor leaves the zero value: `shuttleengines.go:304` builds the hook's engine through `NewFromConfig`, and every other `&Claude{}` literal is in a test that never calls `BatchingNudge`.
  The package doc describes the wait: borne out.
- **Tests.**
  `EditReachingTheTranscriptDuringTheWaitFires`, `ToolUseIDAbsentFromTheTailIsSilentAfterTheBoundedWait` and the two late rows each cover a behavior no existing row covers, because every existing row holds the call in the transcript.
  `wantSleeps` at its zero value on the existing rows is not a duplicate assertion; it is the check that those rows never wait.
  The `CorruptStateIsReportedAndExitsZero` substring `the batching nudge state` still matches the new error text and still pins the error path.
- **Comments and docs.**
  doc.go names the tick and attempts by constant, as the repository's comment standard asks; the one derived figure is in F5.

## Rejected

- Affected-set NIT on `docs/overview.md:678`: that paragraph describes what the hook does and what it judges, not how it reads the transcript, and the wait changes none of the outcomes it lists, so the diff leaves nothing in it wrong.
- Correctness finding that a parallel message could be judged with only some of its lines written: the summary's probe shows a message's lines land together once the model finishes, the same exposure existed for Bash calls before this change, and no evidence in the diff or the tests shows a partial write.
- Tests NIT on the loosened `wantStderr` in `hookbatching_test.go:83`: the substring is the new error's own text, the neighbouring rows assert by substring in the same way, and the error path stays pinned.
