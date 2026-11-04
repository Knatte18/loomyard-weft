```yaml
lens: code-tests
outcome: reported
```

Task: lens code-tests. For each behaviour the diff adds or changes, I name the test that pins it, or "none".

**Per behaviour: pinning test**

- **Read-only Bash lookup counts in the lookup chain.** Pinned by `ReadOnlyBashCommandsCountAsLookups` (`batchingnudge_test.go:194`). It fails if Bash is not a lookup in `tailCall.lookup`, or if the transcript `command` is not parsed.
- **A non-lookup Bash call breaks the chain.** Pinned by `BashCommandThatIsNoLookupBreaksTheChain` (`:195`). If the call counted as a lookup, the run would be 4 and would fire.
- **The early return for a non-lookup Bash call, before the lock and the transcript read.** None. `BashCallThatIsNoLookupIsSilent` (`:196`) stays silent without the early return too, because `kindOf` of the current message is `kindOther`. The state file is unchanged either way. Nothing observable pins the early return, such as a corrupt state file or a missing transcript with a non-lookup Bash call.
- **`batchingNudgeMatcher` now includes Bash.** None. `TestPrepare_BatchingNudgeHook` (`:495`) compares each entry's matcher to the constant itself, so dropping `toolBash` from the constant still passes.
- **`isReadOnlyLookup` command table.** Mostly pinned by `TestIsReadOnlyLookup` (`bashlookup_test.go:20-67`). The rows it misses are listed under the findings below.
- **A call with `agent_id` reads the agent's own transcript.** Pinned by `SubagentLookupChainFiresFromItsOwnTranscript` (`:229`). The session transcript holds no chain, so the agent file has to be the one read.
- **The context reading comes from the agent's sidechain lines (`latestReading(..., true)`).** Pinned by the same row. If the flag were ignored, the agent file's lines would give no reading and the row would go silent. `SubagentBelowTheContextThresholdByItsOwnReadingIsSilent` (`:231`) is the below-threshold side.
- **`transcript_path` already naming the agent file is read as is.** Pinned by `SubagentTranscriptPathNamingTheAgentFileIsReadAsIs` (`:238`).
- **Per-agent nudge count and cap.** Pinned by `SubagentNudgesCountApartFromTheSession` (`:232`) and `SubagentPastItsOwnCapIsSilent` (`:233`).
- **A fired agent nudge leaves the main count unchanged.** Pinned by the `reflect.DeepEqual` of the whole state after the call (`:352`).
- **A state file with no `agents` key, then an agent nudge.** Pinned. The seeded state has a nil `Agents`, and the after-state is compared in full.
- **An agent edit-chain nudge.** Pinned by `SubagentEditChainFires` (`:230`).
- **The agent-id path guard.** Partly pinned by `AgentIDHoldingAPathSeparatorIsSilent` (`:241`), for `/` only.
- **`tailMessages` sidechain switch, main side.** Pinned by `SidechainLineDoesNotBreakTheChain` and `SidechainLineIsNotCountedInTheChain` (`:198-199`).
- **`tailMessages` sidechain switch, agent side.** None. No row puts a non-sidechain line in an agent transcript and expects it to be skipped.
- **Fixture shape.** The fixtures write `isSidechain: true` on every line of an agent transcript. That matches the real subagent transcripts, which all carry `isSidechain: true`.
- **The `subagentsDirName` and `subagentTranscriptPrefix` constants.** Behaviour-neutral; the existing fork-activity and audit tests cover them.

**Findings**

1. MEDIUM, design. `batchingnudge.go:219-222`, early return for a non-lookup Bash call. No test row fails if it is removed (`batchingnudge_test.go:196`). The summary and the doc comment both claim it returns before the lock and the transcript read. Add one row to `TestBatchingNudge`: a non-lookup Bash call with `corruptState: true` and `want` nil, `wantErr` false. Without the early return it errors.

2. LOW, design. `batchingNudgeMatcher` and `TestPrepare_BatchingNudgeHook` (`:495`). Removing Bash from the matcher fails no test, because the check is tautological. This is Wanted 1's reach condition. Assert in the test that the installed matcher's tool list holds each of `Read`, `Grep`, `Glob`, `Bash`, `Edit` and `Write`, or at least `Bash`.

3. LOW, consistency. `chainTranscript` guard (`batchingnudge.go:188`). Only `/` is pinned (`:241`). The `\`, `.` and `..` branches have no row. Add them as rows of the same table.

4. LOW, consistency. `TestIsReadOnlyLookup` rows missing, as rows of the same table:
   - git subcommands `blame`, `grep`, `ls-files`, `ls-tree`, `rev-parse` and `cat-file`. Deleting one from `gitReadSubcommands` fails nothing. Only `log`, `show`, `diff` and `status` have rows (`bashlookup_test.go:31-34`).
   - find actions `-execdir`, `-ok`, `-okdir`, `-fprint`, `-fprint0`, `-fprintf` and `-fls`. Only `-delete` and `-exec` have rows (`:53-54`).
   - sed `--in-place=…`.
   - `||` and `|&` as segment joins.
   - `&> file`, which should not be a lookup, and `>&2`.
   - `git -C` with a missing directory argument.

5. LOW, consistency. Per-agent `NudgedMessage` is unpinned. `MessageAlreadyNudgedIsSilent` (`:243`) seeds the main count only. A bug that read the main session's `NudgedMessage` for an agent would pass. Add a row seeding `withCount(nudgeAgentID, {Nudges: 1, NudgedMessage: "msg-2"})`.

6. LOW, consistency. `tailMessages` and `latestReading` with `sidechain` true skip main-chain lines (`batchingnudge.go:344`, `usage.go:251`). No agent-transcript row holds an `isSidechain: false` line. The fixture helper's `lineSidechain` override could carry one. Without that row, a regression to `!record.IsSidechain` passes.

7. NIT, consistency. A second agent nudging keeps the first agent's entry (`withCount` uses `maps.Clone`). No row seeds an existing agent entry before another agent nudges. Add it only if you want that pinned.

**Test economy.** I found nothing to flag. `TestIsReadOnlyLookup` tests an unexported helper directly, but `BatchingNudge`'s rows cover only a handful of commands, so the table is the economical place for the many rows. The two `BatchingNudge` rows that use Bash commands pin the integration and do not repeat the table. No test pins static prose, all tests are tier 0 and parallel, and there are no skips.

**Tool calls:** none. The manifest context held everything this lens needed.
