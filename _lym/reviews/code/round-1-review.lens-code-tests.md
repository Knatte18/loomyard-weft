```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test row that pins it, or none.

**Per-behaviour list**

1. A tool call naming the skill's `skills/<name>/SKILL.md` tail loads the skill (`namesPath`, unchanged in behaviour).
   - Pinned by `TestClassifySkillLoad` rows `ReadAfterTheMessage`, `CustomToolAndLocalShellCalls`, `RepeatedLoadMessageRestartsTheCount` and `TruncatedLineIsSkipped` (`skillload_test.go:207-241`).
2. A string `output` that holds the whole SKILL.md loads the skill (`toolOutputText` string branch, `printedFile`).
   - Only `ReadBeforeTheMessage` feeds a string output, and that output sits before the message, so it must not count.
   - In `ReadThroughABuiltPath` the string output `"# testing\n"` is partial, so it must not count either.
   - **No row has a string output after the message that holds the whole file.**
   - Break the string branch so it returns `""` and the whole-file outputs still come only from blocks. Every row still passes.
3. A text-block `output` that holds the whole SKILL.md loads the skill (blocks branch and `joinedText`).
   - Pinned by `ReadThroughABuiltPath`, where `Script completed\nOutput:\n` plus `skillBody("prose")` is two blocks, with the file in the second.
   - The test cannot tell joining the blocks from matching each block alone, because the file sits whole in one block.
   - A file split across blocks is not exercised.
   - Break `joinedText` to return only the last block and the row still passes.
4. A partial output does not load the skill (`printedFile` whole-file match).
   - Pinned by `ReadThroughABuiltPath`, where `scribe:testing` stays Missing because `"# testing\n"` is a strict prefix of the file.
5. An output before the load message does not count (`turn` reset at the message).
   - Pinned by `ReadBeforeTheMessage`, whose output is the whole `testing` body and which expects `scribe:testing` Missing.
6. An output of an unsupported shape (neither string nor blocks), or a malformed output payload, yields `""` and loads nothing (`toolOutputText` failure returns).
   - **None.** No row has an output of another type, for example an object, null or a missing `output` key.
7. A missing or empty SKILL.md never counts as printed (`printedFile` `err != nil || content == ""`).
   - **None.** An empty file would be contained in any output, so the guard matters. No row has an empty file or an unreadable file.
   - This is a minor gap, because `skillPath` only returns files that were stat'ed.
8. Whitespace trimming of the file content before matching (`strings.TrimSpace`).
   - Partly pinned. `skillBody` ends in `"\n"`, and `ReadThroughABuiltPath` has the block text `skillBody("prose")` with the same trailing newline, so the trim is not discriminated.
   - A string output with the trailing newline cut off would be needed to pin it.
9. `function_call_output` and `custom_tool_call_output` are both collected (`toolOutputPayloads`).
   - Both types appear: `function_call_output` in the string rows, `custom_tool_call_output` in the blocks row.
   - Only the blocks row asserts a positive match, so the string kind has no positive case (see item 2).
10. Output collection starts only once a load message was found (`found &&` guard on the output case).
    - Pinned by `ReadBeforeTheMessage`, which has an output before the message.
    - `NoLoadMessage` has no output line, but it returns unverified regardless.
11. A repeated load message drops the earlier outputs along with the earlier calls (`turn = loadTurn{}`).
    - **None.** `RepeatedLoadMessageRestartsTheCount` has no outputs, so only the calls reset is pinned.
    - Resetting only `calls` would pass every row.

**Findings**

- **LOW, scope (coverage gap), `internal/shuttleengine/codexengine/skillload_test.go:207-226`.** No row has a string `output` after the load message holding the whole file. The string branch of `toolOutputText` is covered only by negative uses.
  - Evidence: `rolloutStringOutput` is used only at lines 214 and 223.
  - Fix: add the whole-file string output to a new row, or to the `ReadThroughABuiltPath` output lines.
- **LOW, scope (coverage gap), `skillload_test.go:222`.** Block joining is not discriminated, because the whole file sits in one block. A file split across two blocks would pin `joinedText`.
  - Fix: split the `skillBody("prose")` text across two blocks in the existing row.
- **LOW, scope (coverage gap), `skillload_test.go:233-236`.** `RepeatedLoadMessageRestartsTheCount` has no output before the second message, so the reset of `outputs` is untested.
  - Fix: add a whole-file output between the two messages and expect Missing for that skill.
- **LOW, scope (coverage gap), `skillload.go:336-351`.** No row covers an output that is neither a string nor blocks, which must read as `""` and leave the skill Missing.
  - Fix: add a line with a non-string, non-list `output` to an existing row. Do not add a new test function.

Not findings: the empty-file guard and the trim are gaps only in `printedFile`'s defensive details. They need no row, since `skillPath` can only return an existing regular file and the trim is not at risk.

Tool calls made: none. My context already held the diff, `skillload.go` and `skillload_test.go`.
