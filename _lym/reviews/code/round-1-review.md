---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/shuttleengine/claudeengine/batchingnudge.go:219
    summary: The early return for a non-lookup Bash call, which the summary and the doc comment promise comes before the state lock and the transcript read, has no test row that fails without it.
    fix: Add one row to TestBatchingNudge in batchingnudge_test.go with a non-lookup Bash message as the last message (for example Bash:lym board list), corruptState true, want nil and wantErr false. Without the early return the corrupt state file makes BatchingNudge return an error, so the row fails when the return is removed.
    origin: lens:code-tests
  - id: F2
    severity: LOW
    class: design
    location: internal/shuttleengine/claudeengine/bashlookup.go:82
    summary: gitReads accepts a listed git subcommand with any arguments, so git diff, log or show with --output=<file>, git grep with -O or --open-files-in-pager, and --ext-diff are classed as read-only lookups although they write a file or run a program.
    fix: In gitReads, once the subcommand matches gitReadSubcommands, reject the call when any later argument equals --ext-diff or starts with --output, --open-files-in-pager or -O. Add rows to TestIsReadOnlyLookup in bashlookup_test.go for git diff --output=x, git log --output x, git grep -O less foo and git log --ext-diff, each false. Name these options beside rg --pre in the Bounds section of the summary only if the rejection is left out.
    origin: lens:code-correctness
  - id: F3
    severity: LOW
    class: design
    location: internal/shuttleengine/claudeengine/batchingnudge_test.go:495
    summary: TestPrepare_BatchingNudgeHook compares the installed matcher to the batchingNudgeMatcher constant itself, so dropping Bash from the constant, the reach condition of Wanted 1, fails no test.
    fix: In TestPrepare_BatchingNudgeHook add an assertion that strings.Split(batchingNudgeMatcher, "|") holds each of toolRead, toolGrep, toolGlob, toolBash, toolEdit and toolWrite.
    origin: lens:code-tests
  - id: F4
    severity: LOW
    class: consistency
    location: internal/shuttleengine/claudeengine/template.yaml:19
    summary: The max_nudges comment says it is the most nudges one run's session gets, but the diff caps each subagent and fork separately from the main session.
    fix: Reword the comment on max_nudges in template.yaml to say it is the most nudges the session, and each of its subagents and forks, gets, matching the doc comment of BatchingNudgeConfig.MaxNudges.
    origin: lens:code-affected-set
  - id: F5
    severity: LOW
    class: consistency
    location: internal/shuttleengine/claudeengine/bashlookup_test.go:20
    summary: TestIsReadOnlyLookup lacks rows for most table entries, so deleting a git subcommand other than log, show, diff or status, or a find writing action other than -exec and -delete, fails no test.
    fix: Add rows to TestIsReadOnlyLookup. True rows for git blame, git grep, git ls-files, git ls-tree, git rev-parse and git cat-file, for the join forms a || b and a |& b, and for >&2. False rows for find with each of -execdir, -ok, -okdir, -fprint, -fprint0, -fprintf and -fls, for sed --in-place=.bak, for grep a x &> out.txt, and for git -C with no directory argument.
    origin: lens:code-tests
  - id: F6
    severity: LOW
    class: consistency
    location: internal/shuttleengine/claudeengine/batchingnudge_test.go:241
    summary: The agent id guard in chainTranscript has one row for a slash, so the backslash, dot and double dot branches are unpinned.
    fix: Add rows next to AgentIDHoldingAPathSeparatorIsSilent in TestBatchingNudge with agentID set to a\b, . and .., each wanting no nudge.
    origin: lens:code-tests
  - id: F7
    severity: LOW
    class: consistency
    location: internal/shuttleengine/claudeengine/batchingnudge_test.go:243
    summary: MessageAlreadyNudgedIsSilent seeds only the main session's NudgedMessage, so a bug reading the main session's last nudged message for an agent would pass every row.
    fix: Add a row to TestBatchingNudge with agentID nudgeAgentID, messages lookupChain and before seeding s.withCount(nudgeAgentID, batchingNudgeCount with Nudges 1 and NudgedMessage msg-2), wanting no nudge.
    origin: lens:code-tests
  - id: F8
    severity: LOW
    class: consistency
    location: internal/shuttleengine/claudeengine/batchingnudge_test.go:198
    summary: No row puts a main-chain line in an agent transcript, so the agent side of the sidechain switch in tailMessages and latestReading, which skips such lines, is unpinned.
    fix: Add a row to TestBatchingNudge with agentID nudgeAgentID whose messages include a message the fixture writes as a line with isSidechain false between two lookups of the chain, wanting the nudge only when that line is not counted. Give writeNudgeTranscript a marker entry for a main-chain line, like nudgeSidechain, to write it.
    origin: lens:code-tests
  - id: F9
    severity: NIT
    class: consistency
    location: internal/shuttleengine/claudeengine/template.yaml:15
    summary: The hook-batching help text and the context_tokens comment still describe a session-wide judgment, although a subagent's call is now judged against its own context and transcript.
    fix: In hookbatching.go change "the session's recent messages" in the verb's Long text to the calling chain's recent messages, and in template.yaml say on context_tokens that the size is the calling chain's own context.
    origin: lens:code-affected-set
---

### [MEDIUM] Early return for a non-lookup Bash call is unpinned

**Location:** internal/shuttleengine/claudeengine/batchingnudge.go:219
**Issue:** The hook now starts after every Bash call, so the early return before the lock and the transcript read is the guard that keeps a non-lookup call cheap.
`BashCallThatIsNoLookupIsSilent` stays silent without it, because `kindOf` of the current message is `kindOther` and the state file is unchanged either way.
Nothing a test can observe tells the two apart, so removing the return breaks no row.

### [LOW] gitReads passes git options that write a file or run a program

**Location:** internal/shuttleengine/claudeengine/bashlookup.go:82
**Issue:** Wanted 1 asks for a read-only lookup, and `gitReads` returns true for a listed subcommand with any argument after it.
`git diff --output=<file>` and the same option on `log` and `show` write a file, `git grep -O<cmd>` runs a program, and `--ext-diff` can run an external diff driver.
The hook only classifies the command, so the effect is one miscounted lookup in a chain, not an execution; the summary's Bounds section names `rg --pre` and sed's `w` but not these.

### [LOW] Matcher test cannot fail when Bash leaves the matcher

**Location:** internal/shuttleengine/claudeengine/batchingnudge_test.go:495
**Issue:** The test finds the hook entry by comparing its matcher to the constant, then checks the command, so the constant's content is never asserted.
Dropping `toolBash` from `batchingNudgeMatcher` would turn off the Bash reach of Wanted 1 and pass.

### [LOW] max_nudges comment names the session only

**Location:** internal/shuttleengine/claudeengine/template.yaml:19
**Issue:** The shipped template, which `lym config claude` shows to operators, says the cap is per run's session, while `BatchingNudge` checks `count.Nudges` of the calling agent against `Settings.MaxNudges` at batchingnudge.go:237.

### [LOW] TestIsReadOnlyLookup misses table entries

**Location:** internal/shuttleengine/claudeengine/bashlookup_test.go:20
**Issue:** The table pins `log`, `show`, `diff` and `status` of the git reads and `-delete` and `-exec` of the find actions.
The other entries of `gitReadSubcommands` and `findWritingActions`, the `||` and `|&` joins, `sed --in-place=`, `&> file` and a bare `git -C` have no row, so each can change unnoticed.
This is one gap in how the table was filled, so it is one finding.

### [LOW] Agent id guard pinned for one separator only

**Location:** internal/shuttleengine/claudeengine/batchingnudge_test.go:241
**Issue:** `chainTranscript` refuses `/`, `\`, `.` and `..`; the row `AgentIDHoldingAPathSeparatorIsSilent` uses `nested/agent`, so the other three branches pass any change.

### [LOW] Per-agent last nudged message is unpinned

**Location:** internal/shuttleengine/claudeengine/batchingnudge_test.go:243
**Issue:** The agent's `NudgedMessage` is what keeps the parallel calls of one agent message to one nudge.
`MessageAlreadyNudgedIsSilent` seeds the main count only, and the agent rows seed `Nudges` alone, so reading the wrong count's `NudgedMessage` for an agent passes.

### [LOW] Agent side of the sidechain switch is unpinned

**Location:** internal/shuttleengine/claudeengine/batchingnudge_test.go:198
**Issue:** The main side has two rows for a sidechain line in the session transcript.
No row holds a main-chain line in an agent transcript, so a change of `record.IsSidechain != sidechain` to `!record.IsSidechain` in `tailMessages` or `latestReading` passes.

### [NIT] Help text and context_tokens comment still say session

**Location:** internal/shuttleengine/claudeengine/template.yaml:15
**Issue:** `hookbatching.go` Long text says the verb judges "the session's recent messages", and the `context_tokens` comment describes the size below which the hook stays silent without saying whose context.
For an agent's call both are now that agent's own, which the reading of "session" does not exclude, so this is wording only.

## Rejected

- Lens code-tests finding 7 (a second agent keeps the first agent's entry through `withCount`): `maps.Clone` is the whole mechanism and `SubagentNudgesCountApartFromTheSession` already compares the full state after a nudge; no separate behavior is uncovered.
- `report.md:184` still says the hook skips every call from a subagent or fork: the file is a dated measurement report, which records one run and is not claiming to stay true.
- `&&`, `||` and newline joins, and the git reads `blame`, `grep`, `ls-files`, `ls-tree`, `rev-parse` and `cat-file`, go past the entry's literal list: the entry says a git read "such as" four and a chain of lookups, the summary declares both, and each reads as the listed ones do.
- Brace groups, `cd x && ls`, a `VAR=1` prefix and `#` comments are classed as no lookup: that is the conservative side of the entry's "any other command breaks a chain".
- `$'...'` words are misread as unterminated quotes and so classed as no lookup: the same conservative side.
- The change description's claims on the command table, the never-a-lookup list, the early return, the `agents` map and the `sidechain` switch: each matches the diff; only the early return lacks a test (F1).
- Wanted 3: the diff changes a comment in `breaches.sh` alone; `events.sh` already reads `subagents/*.jsonl` and `events.jq` matches the unchanged nudge lines, so a landed run's fork nudge attachment is still read.
- Real transcript shape for Wanted 2: the sampled subagent transcripts hold `isSidechain` true on every assistant line and the fixtures write the same, so the fixtures follow the real shape.
