```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test row that would fail if it broke, or "none".

**Per behaviour: the test that pins it**

*Question deny*
- **Deny matches `request_user_input` / `_async`, with or without a namespace prefix; other tools and a shell command naming the tool get `{}`:** `TestQuestionDenyCommand_DeniesAQuestionTool` (all four rows).
- **Deny entry opens `hooks.PreToolUse` before the guards:** `TestHookSettings/QuestionDenyBeforeTheGuards`.
- **Deny and its notice only on an autonomous launch:** `TestPrepare_QuestionDenyAndBatchingHook` Autonomous and Interactive rows. The rows check the exact setting and the notice string, so an unwanted deny breaks the containment check.
- **`denyNotice` order, question notice first:** `TestGuardEntries_RenderEveryRowWithItsCodexText` (`denyNotice(true, rows)`).

*Hook wiring and config*
- **Batching hook after the activity hook, creel mail last:** `TestHookSettings/BatchingAfterTheActivityHook` and `/CreelMailAfterTheBatchingHook`.
- **Hook command literal with the `{}` fallback:** `TestHookSettings`, the `batchingNudgeHookCommand` assertion. The fallback is pinned as a string only; it never runs under sh.
- **Prepare seeds the state file with the template block; off writes none and installs no hook:** `TestPrepare_QuestionDenyAndBatchingHook` Autonomous and BatchingOff rows.
- **Template defaults of the codex.yaml block:** `TestLoadConfig_Template`.
- **Each threshold below its least is refused:** `TestLoadConfig_RefusesInvalidValues`, the five Batching rows.
- **An off block is not validated; `context_tokens: 0` is accepted:** none.
- **Codex registered as a `BatchingNudger`, silent with no state file:** `TestBatchingNudger`.
- **Constructor sets the production rollout wait:** `TestNewFromConfig_SetsTheRolloutWait`.

*Nudge chains and early exits*
- **Lookup chain fires; a shorter chain is silent:** `LookupChainFires`, `LookupChainShorterThanTheRunIsSilent`.
- **Payload call joins the latest cell when its item is not yet written:** `CallWhoseItemIsNotWrittenYetCountsInTheLatestCell`.
- **Two-lookup cell is not a single lookup:** `CellOfTwoLookupsIsSilent`.
- **`function_call` ends a lookup chain and an edit chain:** `FunctionCallEndsTheLookupChain`, `FunctionCallEndsTheEditChain`.
- **A non-lookup shell cell ends the chain:** `ShellCellThatIsNoLookupEndsTheChain`.
- **A non-lookup Bash call and a non-Bash/apply_patch tool leave the state and rollout unread:** `ShellCallThatIsNoLookupLeavesTheStateUnread`, `OtherToolLeavesTheStateUnread`.
- **Edit chain fires; a mixed cell ends it:** `EditChainFires`, `CellMixingAnEditAndALookupEndsTheEditChain`.

*Holds and state*
- **Context threshold and an unknown context reading:** `BelowTheContextThresholdIsSilent`, `NoContextReadingIsSilent`.
- **No state file is silent; corrupt state and malformed input are errors:** `NoStateFileIsSilent`, `CorruptStateIsAnError`, `MalformedInputIsAnError`.
- **Cap, already-nudged cell, per-`transcript_path` keying:** `ThreadPastItsCapIsSilent`, `CellAlreadyNudgedIsSilent`, `AnotherThreadAtItsCapLeavesThisThreadsCountItsOwn`.
- **Count and `NudgedCell` written on a nudge; state unchanged on a silent call:** the state assertions at `batchingnudge_test.go:300` and `:318`.
- **Output is `additionalContext` plus `systemMessage`:** the assertion at `batchingnudge_test.go:315`.
- **Missing `transcript_path` or `tool_use_id` is silent:** none.

*Read-slice and read-again*
- **Read of a short file fires; a long file read once is silent:** `LineRangeReadOfAShortFileFires`, `LineRangeReadOfALongFileReadFirstIsSilent`.
- **Read-again against mtime:** `…ReadEarlierAndUnchangedFires`, `…ChangedSinceItsReadIsSilent`.
- **Each path resolved against its own call's cwd (`file://` form):** `EarlierReadInAnotherDirectoryIsSilent`, `EarlierReadOfTheSameFileFromAnotherDirectoryFires`.
- **Item wait and the bound:** `LineRangeReadWaitsForItsItem`, `LineRangeReadWithoutItsItemAtTheBoundIsSilent`; the zero-sleep rows pin that non-slice calls do not wait.

*Rollout reading*
- **An unreadable or missing rollout gives no nudge and no error:** none.
- **Breach switches `lookup_chain`, `read_slice`, `edit_chain` turned off:** none.
- **`shellCommandOf` on an argv without `-c`/`-lc`:** none.
- **`localPath` on a plain-path cwd:** none.
- **`resolvePath` on an absolute path, and on a relative path with an empty cwd:** none.
- **An item completing outside every open cell is skipped:** none.
- **`parseRolloutEvents` / `latestContext` split:** the existing `ContextTokens` tests plus the context rows above.

*Moved and shared code*
- **`IsReadOnlyLookup` moved and exported:** `TestIsReadOnlyLookup`.
- **`LineRangeFile`:** `TestLineRangeFile`. The separate-argument `--lines N` form has no row.
- **`ReadFileTail`:** `TestReadFileTail`.
- **`CountLinesUpTo` count and unterminated last line:** `TestCountLinesUpTo`. Stopping at the limit has no row.
- **Seam-scan exemption for `sharedPackages`:** `TestProviderSeamImportRule/ImportOfASharedPackageFromAnEngine`.

**Findings**

1. **LOW, design (test coverage).** `BatchingNudge` / `awaitRollout`, `internal/shuttleengine/codexengine/batchingnudge.go:471-473`. Nothing covers the documented "a rollout that cannot be read gets no nudge and no error" (doc at `batchingnudge.go:402`). Every `TestBatchingNudge` row writes the rollout (`batchingnudge_test.go:224`). `TestBatchingNudger`'s `/absent.jsonl` returns before the rollout is read because it has no state file. Fix: add a row whose `transcript_path` names a missing file, with a state file present.

2. **LOW, design (test coverage).** `judgeBatching`, `batchingnudge.go:502, 507 (via 425), 513`. No row turns off `LookupChain`, `ReadSlice` or `EditChain`; the fixture settings at `batchingnudge_test.go:127` have all three on. Dropping any switch check passes every test. Fix: add a per-row settings override and one off-switch row per breach.

3. **LOW, design (test coverage).** `validateBatchingNudge`, `batchingnudge.go:61-63` and the bound at `:68`. No test shows that an off block skips validation, or that `context_tokens: 0` is accepted. `TestLoadConfig_RefusesInvalidValues` (`prepare_test.go:1027-1031`) has refusal rows only, all on the template's `enabled: true`. Fix: add accepting rows, such as `enabled: false` with `lookup_run: 0`, and `context_tokens: 0`.

4. **LOW, design (test coverage).** `CountLinesUpTo`, `internal/shuttleengine/filetail.go:1178`. The test's doc (`filetail_test.go:1255`) says it pins that "a count stops at the limit", but every row is at or under the limit. A loop that ignores `limit` passes. Fix: add a row with more lines than the limit, expecting the limit.

5. **LOW, design (test coverage).** `rolloutCalls`, `shellCommandOf`, `localPath`, `resolvePath`, `batchingnudge.go:271-289, 297-322`. Several parsing branches have no row:
   - an item completing outside every open cell, which is skipped;
   - an argv without `-c`/`-lc`, which is joined;
   - a plain-path cwd that is not a `file://` URL;
   - an absolute `parsed_cmd` path, and a relative one with an empty cwd.

   The fixture always writes `/bin/bash -lc`, a `file://` cwd, relative reads, and items inside the open cell (`batchingnudge_test.go:76-77, 101-106`). Fix: add `TestBatchingNudge` rows, or fixture knobs, for these shapes.

6. **NIT, design (test coverage).** `BatchingNudge`, `batchingnudge.go:409-411`. A payload with no `transcript_path` or no `tool_use_id` is silent, and no row covers it. Fix: add a row with a payload that omits `tool_use_id`.

7. **NIT, design (test coverage).** `countedLinesFile`, `internal/shuttleengine/bashlookup/bashlookup.go:122-127`. The separate-argument `--lines N` form is accepted, and the function's doc names it, but `TestLineRangeFile` has rows only for `--lines=N`. Fix: add a `head --lines 50 a.go` row.

8. **LOW, consistency (test economy).** The redundancy report lists three tests as contained by `TestPrepare_CreelHooksInAHub`. That test's expected settings come from `hookSettings` itself (`prepare_test.go:493`), it checks only the Stop and PostToolUse settings, and it never runs a hook command. So each of the three pins something coverage cannot see, and each should get `//testtiming:keep &lt;reason&gt;` above its `func Test…` line rather than be folded:
   - `TestHookSettings` (`command_test.go:84`) pins the exact setting strings and order, plus the literal batching hook command.
   - `TestQuestionDenyCommand_DeniesAQuestionTool` (`guards_run_test.go:806`) runs the grep pattern under sh, including the shell-command false-positive row.
   - `TestGuardEntries_RenderEveryRowWithItsCodexText` (`guards_test.go:57`) pins the literal deny JSON and the deny notice order.

Tool calls:
- One Bash `grep` for `TestPrepare_CreelHooksInAHub`, the head of `TestGuardEntries_RenderEveryRowWithItsCodexText` and the existing `//testtiming:keep` markers. My context held only the diff hunks of these tests, not their full bodies, which I needed to compare assertions for finding 8.
