---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md:705
    summary: The overview's shuttle section still describes the batching nudge and the question-tool deny as Claude-only; the Codex bullet at line 713 names only the Bash shell-guard PreToolUse hooks.
    fix: In docs/overview.md, extend the Codex bullet at line 713 to state that an autonomous Codex launch also denies request_user_input and request_user_input_async through a `*`-matcher PreToolUse entry, and that while codex.yaml's batching_nudge block is on a `*`-matcher PostToolUse hook runs `lym shuttle hook-batching codex`, judging the thread's exec cells by Claude's rules with counts keyed by transcript_path and handing the line over as additional context, never a block.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/shuttleengine/codexengine/command_test.go:71
    summary: The three tests the redundancy report lists as contained by TestPrepare_CreelHooksInAHub each pin what that test cannot see, and none carries a keep marker.
    fix: Add a `//testtiming:keep <reason>` line directly above each `func Test…` line. Above TestHookSettings in command_test.go, the reason is that it pins the exact hook setting strings and their order and the literal batching hook command; TestPrepare_CreelHooksInAHub derives its expectation from hookSettings itself. Above TestQuestionDenyCommand_DeniesAQuestionTool in guards_run_test.go, the reason is that it runs the deny's grep pattern under sh, including the shell-command false-positive row. Above TestGuardEntries_RenderEveryRowWithItsCodexText in guards_test.go, the reason is that it pins the literal deny JSON and the deny notice order, question notice first.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/codexengine/batchingnudge.go:305
    summary: localPath turns a Windows file URL such as file:///C:/x into \C:\x, which carries no volume, so every read path resolves to a file that cannot be read and the read-slice breach never fires on Windows.
    fix: In localPath, after url.Parse succeeds with scheme file, strip the leading slash when the parsed path is a drive path (a slash, a letter, then a colon, such as /C:/x), then apply filepath.FromSlash. State the drive-letter case in localPath's doc comment.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/codexengine/batchingnudge_test.go:161
    summary: No TestBatchingNudge row covers a rollout that cannot be read, which BatchingNudge's doc promises gets no nudge and no error, nor a payload missing transcript_path or tool_use_id.
    fix: Add two rows to TestBatchingNudge. In the first, the hook input's transcript_path names a file that does not exist while the state file is seeded, and the row wants no nudge, no error and an unchanged state file. The second passes a payload that omits tool_use_id and wants no nudge. Give the table a field that overrides transcript_path for the first row.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/codexengine/batchingnudge.go:500
    summary: No test turns off lookup_chain, read_slice or edit_chain, so dropping any switch check in judgeBatching or BatchingNudge passes every test.
    fix: Give TestBatchingNudge's table a settings-override field and add one row per switch. Each row repeats a firing row (LookupChainFires, LineRangeReadOfAShortFileFires, EditChainFires) with that row's switch off and wants no nudge.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/codexengine/batchingnudge.go:60
    summary: No test shows that validateBatchingNudge skips an off block or accepts context_tokens 0; TestLoadConfig_RefusesInvalidValues holds refusal rows only.
    fix: Add a test in prepare_test.go beside TestLoadConfig_RefusesInvalidValues, or rows in an existing accepting LoadConfig test, that load a codex.yaml layer of `batching_nudge` with `enabled false` and `lookup_run 0`, and one of `context_tokens 0` with the block on, each loading without error.
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/filetail_test.go:1
    summary: TestCountLinesUpTo's doc says it pins that a count stops at the limit, but every row is at or under the limit, so a loop that ignores the limit passes.
    fix: Add a row to TestCountLinesUpTo whose content holds more lines than its limit, such as five lines with limit 2, and want 2.
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/codexengine/batchingnudge.go:233
    summary: The rollout-reading branches the fixture never writes have no row; these are an item completing outside every open cell, an argv without -c or -lc, a plain-path cwd, and an absolute parsed_cmd path.
    fix: Extend the TestBatchingNudge fixture so a fixtureNested can set a plain-path cwd, an absolute read path and a non-shell argv, and so a fixture can place an item after its cell's output. Add rows that pin each one. A plain cwd and an absolute read path still fire the read-again nudge. A non-shell argv lookup still counts toward lookup_run. An item after its cell's output joins no cell.
  - id: F9
    severity: NIT
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/bashlookup/bashlookup_test.go:208
    summary: countedLinesFile accepts the separate-argument --lines N form its doc names, but TestLineRangeFile has a row only for --lines=N.
    fix: Add the row {"HeadLongCountSeparate", "head --lines 50 a.go", "a.go", true} to TestLineRangeFile.
---

### [LOW] The overview still describes the nudge and the question deny as Claude-only

**Location:** `docs/overview.md:705`, `docs/overview.md:713`

**Issue:** Line 705 describes the batching nudge under `claude.yaml` alone.
The Codex bullet at line 713 lists only the shell-guard `PreToolUse` hooks matching `Bash`.
The diff adds a `*`-matcher question deny on autonomous Codex launches and a Codex `batching_nudge` `PostToolUse` hook, and it touched this file only for the package list at line 639.

### [LOW] Contained tests need keep markers

**Location:** `internal/shuttleengine/codexengine/command_test.go:71`, `guards_run_test.go:18`, `guards_test.go:57`

**Issue:** The redundancy report lists `TestHookSettings`, `TestQuestionDenyCommand_DeniesAQuestionTool` and `TestGuardEntries_RenderEveryRowWithItsCodexText` as contained by `TestPrepare_CreelHooksInAHub`.
That test builds its expectation by calling `hookSettings` (`prepare_test.go:493`), so it checks only that `Prepare` forwards the settings.
It never pins their content or order, never runs a hook under sh, and never reads the deny notice.
Each candidate pins something coverage cannot see, so the rubric asks for a keep marker, not a fold.

### [LOW] localPath breaks a Windows file URL

**Location:** `internal/shuttleengine/codexengine/batchingnudge.go:305`

**Issue:** `url.Parse("file:///C:/x").Path` is `/C:/x`, and `filepath.FromSlash` makes it `\C:\x`, which has no volume.
`resolvePath` then joins read paths onto it, `CountLinesUpTo` and `os.Stat` fail, and `sliceBreach` returns no breach.
So on Windows the read-slice nudge never fires and nothing reports why.

### [LOW] An unreadable rollout and missing payload ids are untested

**Location:** `internal/shuttleengine/codexengine/batchingnudge_test.go:161`

**Issue:** Every row writes the rollout at `batchingnudge_test.go:224`.
`TestBatchingNudger`'s `/absent.jsonl` returns before the rollout is read, because it has no state file.
The doc's promise at `batchingnudge.go:402` and the early return at `:409-411` therefore have no row that fails when either breaks.

### [LOW] The breach switches are untested

**Location:** `internal/shuttleengine/codexengine/batchingnudge.go:500`

**Issue:** The fixture settings at `batchingnudge_test.go:127` turn every switch on.
Removing the `LookupChain`, `ReadSlice` (`:425`) or `EditChain` check passes every test.

### [LOW] validateBatchingNudge's accepting paths are untested

**Location:** `internal/shuttleengine/codexengine/batchingnudge.go:60`

**Issue:** The off-block skip at `:61-63` and the `context_tokens` floor of 0 at `:68` have no accepting row.
A change that validates an off block, or refuses 0, passes `TestLoadConfig_RefusesInvalidValues` and `TestLoadConfig_Template`.

### [LOW] TestCountLinesUpTo does not pin the limit it claims

**Location:** `internal/shuttleengine/filetail_test.go:1`

**Issue:** The doc of `TestCountLinesUpTo` says a count stops at the limit.
Every row's content is at or under its limit of 10, so the stop at `filetail.go` `for lines < limit` is unpinned.

### [LOW] Rollout-reading branches the fixture never writes

**Location:** `internal/shuttleengine/codexengine/batchingnudge.go:233`

**Issue:** The fixture always writes `/bin/bash -lc`, a `file://` cwd, relative read paths and items inside the open cell (`batchingnudge_test.go:76-77, 101-106`).
These branches therefore have no row:
- `rolloutCalls` skipping an item outside every open cell;
- `shellCommandOf` joining an argv without a shell flag;
- `localPath` passing a plain path through;
- `resolvePath` keeping an absolute path.

### [NIT] The separate-argument --lines form has no row

**Location:** `internal/shuttleengine/bashlookup/bashlookup_test.go:208`

**Issue:** `countedLinesFile` (`bashlookup.go:122-127`) accepts `--lines N`, and its doc names that form.
No `TestLineRangeFile` row uses it.

## Ground truths checked

- The gate's tier0 run over the touched packages passed: 2552 passed, 0 failed.
- The 2026-11-16 spike rollout's `CommandExecution` items have the shapes `rolloutCalls` decodes: an argv `command`, a `file://` `cwd`, and `parsed_cmd` read paths as written.
- `OpenLaunch` does not call `hookSettings`, so an opened Codex session is unchanged and `docs/overview.md:644` still holds.
- Every goal point traces to a change and every change to a goal point, so the review raised no scope finding.

## Rejected

- affected-set F2, the `hookbatching.go` help text "judges the call and the calling chain's recent messages": the wording is engine-neutral and still true of a Codex thread's calls, so nothing there is wrong.
- affected-set F3, commit `f381f428` saying the scan exempts packages that hold no engine: the pattern background and the change description both state the explicit `sharedPackages` list. A commit body paraphrase that the diff carries correctly is no defect the diff can fix.
- code-correctness's unfiled premise, that Codex may write a call's item only after PostToolUse returns: the decision record sets the bounded wait and its no-nudge outcome (D12), so this is no defect of the diff.
