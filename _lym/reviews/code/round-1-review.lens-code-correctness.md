```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each function and branch the diff adds or changes with concrete inputs.

**Per-function list**

- `BatchingNudge`, malformed JSON payload: returns a wrapped error. The verb prints it to stderr and exits 0. Correct.
- `BatchingNudge`, no `agent_id`, empty `transcript_path` or empty `tool_use_id`: the first two are silent, the third returns silent. Correct.
- `BatchingNudge`, state file absent: `os.Stat` returns not-exist, so it is silent. Correct. Stat fails for another reason: error, exit 0. Correct.
- `BatchingNudge`, `UpdateJSON` callback returns `errNoNudge`: no write happens, and the `errors.Is` check maps it to silent. Correct.
- `BatchingNudge`, corrupt state file: `UpdateJSON` aborts before the callback and the error is returned. Correct.
- `BatchingNudge`, `ContextTokens` 0: `Tokens < 0` is never true, so it always fires. Acceptable, since the validator allows 0 deliberately.
- `BatchingNudge`, state write fails after a line is composed: the error comes back and no nudge is emitted. Count and message id stay consistent. Correct.
- `readTranscriptTail`, file smaller than the budget: reads whole file. Correct.
- `readTranscriptTail`, file larger than the budget: drops the cut first line. Correct.
- `readTranscriptTail`, window with no newline: returns an empty tail, so `latestReading` finds nothing and it is silent. Correct.
- `readTranscriptTail`, `io.EOF` on a short read: ignored. Correct.
- `latestReading` (unchanged code, reused here): sums input, cache-creation and cache-read tokens, skips sidechain lines, and handles `compact_boundary`. The threshold therefore compares against full context, as the Design says. Correct.
- `tailMessages`, one message id spread over several assistant lines (thinking, text, tool_use): calls merge under one message and `at` is the first line's time. Correct.
- `tailMessages`, sidechain line, unparsable line, empty id: all skipped. Correct.
- `tailMessages`, fractional-second or `Z` timestamp: parses with `RFC3339Nano`. A bad timestamp leaves `at` zero, which `sliceBreach` handles by returning no breach. Correct.
- `kindOf`, text-only or thinking-only message (no `tool_use`): `kindOther`, which breaks a run. Correct against the record. A transcript line interleaved between assistant lines can't change that, because user lines are skipped.
- `kindOf`, one Read, Grep or Glob call: `kindLookup`. Two Reads: falls through to the edit loop and returns `kindOther`. Correct.
- `kindOf`, Edit and Write mixed: `kindEdits`. Edit plus Bash: `kindOther`. Correct.
- `runEndingAt` and `judgeBatching`, three single-lookup messages with `LookupRun` 3: fires with run 3. Two lookups after a Bash message: silent. Correct.
- `judgeBatching`, a parallel message of N calls: it counts as one message and the once-per-message guard gives one nudge. Correct.
- `judgeBatching`, lookup chain fires on the current call: it shadows the read slice. That is by design and documented.
- `sliced`, offset 0 or 1 with no limit: not sliced. Offset greater than 1, or any limit: sliced. Correct.
- `sliceBreach`, short file: breach. File with exactly `SliceLines` lines: not under, so no breach. Correct.
- `sliceBreach`, an earlier Read at time T and the file's mtime after T: silent. mtime equal to or before T: breach. The earlier Read's own timestamp is its first line, which precedes the Read executing. A modification between that timestamp and the actual read therefore reads as "changed" and stays silent, the safe direction. Correct.
- `sliceBreach`, no earlier Read of the path in the tail: silent. Correct.
- `countLinesUpTo`, empty file: 0. Unterminated last line: counted. Early exit at the limit: correct. A directory: the read errors and there is no breach. Correct.
- `Prepare`, `ValidateBatchingNudge` runs before any artifact is written. State is seeded only when `Enabled`. Correct.
- `Prepare`, state-seed failure after `settings.json` is written: returns an error with partial artifacts. This is the same pattern as the neighbouring writes. Not a finding.
- `ValidateBatchingNudge`, disabled block with bad values: skipped. Enabled with `lookup_run` 0: refused, naming key and `lym config claude`. Correct.
- `buildSettings`, state path from `path.Join(path.Dir(eventsPathPosix), …)`: it is the same directory as `runDir` while the events file is in `runDir`. The test pins the equality on non-Windows. Correct.
- `hookBatchingCmd` and `hookBatching`, unknown engine: error to stderr, exit 0. No output: nothing printed. A nudge: output plus newline. The `PersistentPreRunE` skip matches `cmd.Name()`. Correct.
- `hookBatchingCmd`, wrong argument count: `cobra.ExactArgs(2)` returns a non-zero exit. The hook command is generated with exactly two arguments and ends in `; true`, so this is unreachable in practice. Not a finding.

**Findings**

1. **LOW, design, `internal/shuttleengine/claudeengine/batchingnudge.go:331-346`.** `kindOf` classes a message only by the calls already in the transcript. A parallel message of two Reads whose later `tool_use` line is not yet written when the first call's hook runs reads as one single lookup, which nudges the very behavior the rule asks for. `claudeengine/doc.go` documents this as a bound, and the test fixtures always carry the whole message, so it is unpinned and unmitigated. Mitigating it would need a settle step or a check on `stop_reason`, and I did not verify how Claude Code writes these lines.
2. **LOW, design, `internal/shuttleengine/claudeengine/batchingnudge.go:383-407`.** The "already read" test accepts any earlier Read of the path, including an earlier sliced Read. Two slices of a long file therefore fire "a file you already read that has not changed since" although the file was never read whole. The nudge text is wrong for that case, and the Design says "already read", so this is a judgment call.
3. **LOW, design, `internal/shuttleengine/claudeengine/claudeengine.go:223-227`.** I could not establish whether a resumed run calls `Prepare` on a run directory that already has a nudge state file. `Prepare` has one production caller, `internal/shuttleengine/run.go:612`. If it does, `writeBatchingNudgeState` reseeds with `Nudges` 0 and clears `NudgedMessage`, which resets the cap for a resumed session. Unverified; I did not read `run.go`'s resume path.

**Tool calls**
1. `grep` for non-test `.Prepare(` callers: to see whether a resume re-runs `Prepare`. My context held only the definition.
