```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I name the test that pins it, or none.

**Per behaviour: pinning test**

| Behaviour | Pinned by |
|---|---|
| Single-lookup chain fires at `lookup_run`, using Read, Grep and Glob | `TestBatchingNudge/LookupChainFires` |
| A shorter run, or a run broken by another tool, is silent | `LookupChainShorterThanTheRunIsSilent` |
| A batched current message is silent | `BatchingNudgeCurrentMessage` row `BatchedCurrentMessageIsSilent` |
| Context threshold from the transcript usage | `BelowTheContextThresholdIsSilent`. The above-threshold rows use `input_tokens` only; the cache-token summation is not exercised here. |
| Read slice of a file under `slice_lines` fires | `SlicedReadOfAShortFileFires` |
| Read slice of a long, unread file is silent | `SlicedReadOfAnUnreadLongFileIsSilent` |
| Read slice of a file already read and unchanged fires | `SlicedReadOfAnUnchangedFileReadBeforeFires` |
| Read slice of a file changed since the earlier read is silent | `SlicedReadOfAFileChangedSinceIsSilent` |
| Edit-only chain fires, with Edit and Write | `EditChainFires` |
| An edit after a lookup is silent | `EditAfterALookupIsSilent` |
| Subagent call is silent | `SubagentCallIsSilent` |
| Nudge cap | `NudgeCapReachedIsSilent` |
| Once-per-message guard | `MessageAlreadyNudgedIsSilent` |
| `lookup_chain` switch off | `SwitchedOffBreachIsSilent` |
| No state file: silent, nothing written | `NoStateFileIsSilent` |
| Fired nudge counts and records the message; a silent call leaves state unchanged | the state assertion in every row |
| `Prepare` installs the hook and seeds state; off installs neither; a bad threshold refuses before any artifact | `TestPrepare_BatchingNudgeHook` |
| Template defaults | `TestLoadConfig` |
| `CheckLoad` refuses a bad threshold | `TestConfigModules_ClaudeCheckLoad/BatchingRunBelowOne` |
| `BatchingNudger` registry lookup | `TestBatchingNudger` |
| Shipped batching rule wording | `store_test.go` `ShippedEntriesArePinned`. The plan-edits wording is static prose, so exempt. |

**Findings (behaviours with no row)**

- **F-T1, MEDIUM, design.** The `hook-batching` verb has no test: `internal/shuttlecli/hookbatching.go:20-63`, with the resolution skip in `cli.go:62`.
  - Nothing pins exit code 0 on an unknown engine, an unreadable stdin or an engine error.
  - Nothing pins the stdout line or the nudge, or that the verb runs outside a hub or git worktree.
  - `TestBatchingNudger` covers only the registry.
  - Add a tier0 test in `shuttlecli` that runs the verb through the cobra command with an unknown engine and with a fixture state and transcript.
- **F-T2, MEDIUM, design.** The real transcript format is untested: `batchingnudge.go:253-282` (`tailMessages`).
  - Claude Code writes one JSONL line per content block, all sharing one `message.id`.
  - The fixture `writeNudgeTranscript` puts every call of a message on one line, so the merge of split lines is never exercised.
  - The key behaviour "parallel calls of one message count once" is therefore only half pinned.
  - Add a fixture row where one message's calls span two lines. Add a sidechain line and a line with no message id that must be skipped.
- **F-T3, MEDIUM, design.** `readTranscriptTail` (`batchingnudge.go:209-235`) has no test of the tail window.
  - Untested cases are a file larger than `batchingTailBytes` dropping its cut first line, a window with no newline returning nil, and a smaller file read whole.
  - `batchingTailBytes` is a constant, so a test through `BatchingNudge` cannot shrink it.
  - It needs an `export_test.go` hook or a budget parameter passed through, or a transcript over 4 MiB.
- **F-T4, LOW, design.** `sliced()` (`batchingnudge.go:146-151`) has no row for its boundaries.
  - Every sliced row sets both `offset=10` and `limit=5`.
  - Untested: a limit alone fires, `offset: 1` alone is not a slice, and a plain full Read is not a slice.
  - Add rows in `TestBatchingNudge`.
- **F-T5, LOW, design.** The switches `read_slice` and `edit_chain` have no off row.
  - Only `lookup_chain` is tested off, so a wrong `settings.ReadSlice` or `settings.EditChain` guard in `judgeBatching` (`batchingnudge.go:367-372`) would pass.
- **F-T6, LOW, design.** Failure paths of `BatchingNudge` (`batchingnudge.go:158-204`) are untested.
  - Malformed payload JSON returns an error.
  - An empty `transcript_path` or `tool_use_id` is silent.
  - An unreadable transcript is silent with no error.
  - A corrupt state file returns an error.
  - A `tool_use_id` absent from the tail is silent.
  - The `TestBatchingNudger` `absent.jsonl` case reaches only the no-state-file exit.
- **F-T7, LOW, design.** `ValidateBatchingNudge` (`batchingnudge.go:50-72`) is pinned only by a `lookup_run: 0` row.
  - Untested: a negative `context_tokens` refused, `context_tokens: 0` accepted, an `enabled: false` block with bad values accepted, and the other keys each refused.
  - Fold these into the `TestPrepare_BatchingNudgeHook` refusal rows as a table.
- **F-T8, LOW, design.** Ordering of breaches (`batchingnudge.go:358-378`) is unpinned.
  - No row has a lookup chain and a read slice true at once, so the documented order (lookup, then slice, then edit) could be swapped unnoticed.
- **F-T9, LOW, design.** The edit-only kind with several parallel calls in one message is untested.
  - `kindOf` (`batchingnudge.go:331-347`) accepts several Edit or Write calls per message.
  - Every `EditChainFires` message holds one call.
  - There is also no row with a mixed Edit and Read message, which must be `kindOther`.
- **F-T10, LOW, design.** The cache-token summation is not exercised in `BatchingNudge`.
  - `latestReading` is shared existing code, but the fixture usage carries only `input_tokens`.
  - A fixture row with `cache_read_input_tokens` over the threshold would pin that the hook reads the full context.
- **F-T11, LOW, design.** The slice-breach threshold edge is untested: a file of exactly `slice_lines` lines is not short.
  - `countLinesUpTo` is also untested for a last line without a newline (`batchingnudge.go:388,409-435`).
  - The fixtures use 5 and 40 lines with a 20 limit.

Not a finding: the hook's order relative to the creel mail hook in `settings.go:378-388`. `TestPrepare_BatchingNudgeHook` filters by matcher and does not pin order, which the diff documents only in a comment.

**Tool calls**
- None made. The manifest already held the diff, tests, `latestReading` and `state.UpdateJSON`.
