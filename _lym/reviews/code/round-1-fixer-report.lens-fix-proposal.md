```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer applied as proposed (F1–F5) and build cases where they are wrong.

**MEDIUM — F4, `doc.go` after "waits only while the tail's first reading…" and the `awaitCallInTranscript` doc comment**
- **Wrong text:** "That reading is the previous message's, so the one call whose message first reaches `context_tokens` is left silent."
- **Case:** the text holds only when the call's message is not yet in the tail at attempt 0.
- **Evidence:** when the transcript already holds the call's message at attempt 0, `latestReading(tail, sidechain)` returns that message's own usage. That reading is at or above the threshold, so the check passes and the call is judged. In `TestBatchingNudge`, `LookupChainFires` writes every message and fires at 200000 tokens with no wait.
- **Fix:** the new sentences state an unconditional silence the code does not hold. Condition them, for example "When the call's message is not in the transcript yet, that reading is the previous message's, so …".

**LOW — F5, `batchingnudge.go`, the comment above the wait constants**
- **Over-long line:** the proposed single line is about 131 characters of text. Commit `74ea6b01` adds a lint flagging `//` lines of 120 or more, and `prose` sets the same 120 limit.
- **Fix:** break it at ", and".
- **Dangling "It":** the next line, "It leaves out only the early calls…", lost its antecedent. The plural "The production tick and attempts" now opens the paragraph where "The production wait" did. A reader can't tell what "It" refers to.

**LOW — F3, row `EditReachingTheTranscriptAfterSeveralRereadsFires`**
- **Gap:** it does not pin the size gate, the reason F3 gives for the row. A gate that rereads on every attempt also passes: the call is absent on the grown rereads, and `wantSleeps` is 2 either way.
- **Latent defect:** the extra `nudgeSidechain` messages are sidechain lines. In a transcript written with `chainSidechain` true they would count as agent-chain messages that hold calls, so `lateAfter` is safe only on main-chain rows. No current row combines the two.

**No findings:**
- **F1:** the constructor test fails if either constructor drops `transcriptWait`.
- **F2:** all four rows fail with their defect in place:
  - `NoStateFileIsSilentWithoutWaitingForALateCall` fails if the wait runs before the state read, because `wantSleeps` is 0.
  - `SubagentPastItsOwnCapIsSilentWithoutWaitingForALateCall` fails if the cap check follows the wait.
  - `SubagentWithNoReadingYetIsSilentWithoutWaiting` fails if the first-tail check is removed: it waits once, then stays silent on a run of 1.
  - `SubagentEditReachingTheTranscriptDuringTheWaitFires` fires on the sidechain transcript after one wait.

Tool calls made: none. My context already held every changed file.
