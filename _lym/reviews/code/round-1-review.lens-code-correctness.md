```yaml
lens: code-correctness
outcome: reported
```

Task: review the correctness of the code the diff changes (the code-correctness lens), from the inherited context only.

**Walkthrough (function or branch, case run, verdict)**

- **`BatchingNudge`, state read** (`batchingnudge.go:256`)
  - Missing state file: `ReadJSON` returns `found=false`, so it returns nil, nil. Correct.
  - Corrupt state file: `ReadJSON` wraps `ErrDecode`, so it returns the new "read the batching nudge state" error. Correct, and the `hookbatching_test.go` substring `the batching nudge state` still matches.
  - Missing state directory: `ReadJSON` falls back to an unlocked read. Correct.
  - `MaxNudges` is at least 1 by validation, so a cap of 0 does not arise.
- **`BatchingNudge`, cap check outside the lock** (`:260`)
  - The check uses a stale count, but the closure re-checks the cap and `NudgedMessage` under the write lock (`:271`, `:280`).
  - Two parallel calls of one message: the first nudges and records the message id. The second waits on the lock, sees the same id from its own tail, and stays silent. Correct, still at most one nudge per message.
- **Dropped `current < 0` guard** (`:279-280`)
  - The closure reuses the tail that `awaitCallInTranscript` returned, and it returns that tail only when `messageHolding(...) >= 0`.
  - `tailMessages` is deterministic on the same bytes and the same `sidechain`, so `current >= 0` always holds. The drop is safe.
- **`awaitCallInTranscript`, attempt 0**
  - Missing transcript: returns false at once. Correct.
  - Read error: returns false at once. Correct.
  - Window with no line end: `readTranscriptTail` returns nil, nil, `latestReading` is not Known, and it returns false. Correct.
  - Reading below the threshold: returns false without sleeping. Correct.
  - Call already in the tail: returns the tail with 0 sleeps. Correct.
- **`awaitCallInTranscript`, loop bound**
  - Attempts 0 to `attempts` give `attempts` sleeps, and the test with 3 attempts pins 3 sleeps. Bounded.
- **`awaitCallInTranscript`, later attempts**
  - The size gate rereads on growth and also on truncation or compaction rewrite, since the gate is `!=`.
  - A rewrite at the same size is not reread, which is unreachable for an append-only transcript.
  - A stat or read error mid-wait returns false. Correct.
  - A partial last line fails to unmarshal and is skipped, then the next growth triggers a reread. Correct.
  - `readSize` comes from the stat before the read. If the file grows between the two, the next tick does one redundant reread. Harmless.
- **Closure, final context check** (`:274-277`)
  - The final tail is judged again with the same rule. Correct.
  - A compaction during the wait that lowers the reading gives a correct silence.
- **Constructors** (`claudeengine.go:59` and `:65`)
  - `New` and `NewFromConfig` both set `transcriptWait`.
  - `shuttleengines.BatchingNudger` builds through `NewFromConfig`, so the production hook waits. Correct.
  - Bare `&Claude{}` literals in tests never wait, which is the documented zero value.
- **`CorruptStateIsReportedAndExitsZero` error path**
  - `UpdateJSON` is no longer reached on a corrupt file. The read path reports the error, and the verb still exits 0. Correct.

**Findings**

1. **LOW, design.** `batchingnudge.go:319-322`.
   - The context threshold on the first tail is read at attempt 0, when the call's own message is usually not yet written.
   - The reading is therefore the previous message's usage, which is lower than the usage of the message holding the call.
   - A chain whose context crosses `context_tokens` in the call's own message is wrongly silenced, because the wait never starts. The old code judged on the final tail.
   - A subagent's first message has no sidechain reading yet, so it is silent. That case is harmless, since a chain needs at least two messages.
   - The closure re-check on the final tail can only silence, never rescue, so the two readings disagree only in the silencing direction.
   - Evidence: `usage.go:253-258` takes the latest assistant line, and `batchingnudge.go:319-322` runs only at `attempt == 0`.

2. **LOW, design, low confidence.** `batchingnudge.go:324-325`.
   - The wait returns as soon as any line holds the call.
   - If Claude Code writes one line per content block non-atomically, a parallel message can be judged with only some of its calls present. `kindOf` then sees a partial call set, so a message that really holds two calls could be classed as a single lookup, or as edits only, and fire a wrong nudge.
   - The wait makes the lookup happen exactly while the message is being written.
   - `summary.md:15-16` suggests the lines land together after the model finishes, so this may not occur. I did not verify it.

No other defect found in the lens.

**Tool calls:** none. The manifest already held every definition and call site the walkthrough needed.
