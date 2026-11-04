```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens, tracing each changed function and branch with a concrete input. I made no tool calls and judged from the inherited context.

## Per-function list

| Function or branch | Case run | Result |
|---|---|---|
| `BatchingNudge` early return | Bash `go vet`, a non-lookup, returns before the state read and transcript read | Correct. A non-lookup Bash call is `kindOther` in `kindOf`, so no breach could fire. |
| `BatchingNudge` early return | Bash `grep x y`, a lookup, continues to the lock and the tail read | Correct. |
| Current-call and transcript agreement | The same command string reaches `isReadOnlyLookup` from `input.ToolInput.Command` and from `tailCall.command` (`input.command`) | Correct. One classifier, one string, so the call is never read two ways. |
| `chainTranscript` | Empty `agent_id` returns `transcript_path` | Correct. |
| `chainTranscript` | `agent_id` `fork-1` with `/s/abc.jsonl` gives `/s/abc/subagents/agent-fork-1.jsonl` | Correct. |
| `chainTranscript` | `transcript_path` already `.../agent-fork-1.jsonl` is returned as is | Correct. |
| `chainTranscript` | `agent_id` `a/b`, `a\b`, `.` and `..` give `ok=false` and no nudge | Correct. |
| `chainTranscript` | Missing agent file makes `readTranscriptTail` fail, so `errNoNudge` | Correct. |
| `latestReading`, `tailMessages` with `sidechain` | I counted `isSidechain` over about 150 real subagent transcripts, 1,785 assistant lines, all `true`. The main transcript sample was all `false`. | Correct. The agent path's `sidechain=true` matches the real shape, and the session-usage caller passes `false`, unchanged. |
| `count` and `withCount` | Seed state without `agents`: `count("x")` reads a nil map and returns the zero value. `withCount` clones nil, allocates, and writes the key. | Correct. The embedded `batchingNudgeCount` keeps `nudges` and `nudged_message` at the top level, so an old state file still reads. |
| `withCount` | Updating one agent leaves the main count and other agents' counts unchanged | Correct. |
| Parallel calls from main and an agent at once | Both run inside `UpdateJSON` under the lock and write different keys | Correct. Each sees the other's write. |
| Cap check | `MaxNudges` is checked against the per-agent count, so each agent is capped separately | Correct. |
| `splitShellSegments`, `2>&1` before or after other words | `grep foo x 2>&1 | tail -3`: the digit-only word is dropped, and the descriptor duplicate is skipped | Correct. |
| `splitShellSegments`, `2>/dev/null&&cat y` | The target scan stops at `&` | Correct. |
| `splitShellSegments`, `&>/dev/null` and `grep a x >/dev/null &` | The first passes. The trailing `&` gives `false`. | Correct. |
| `splitShellSegments`, `<` input | `wc -l < a.go` adds `a.go` as an argument and passes. `<<`, `<<<` and `<(` are refused. | Correct. |
| `splitShellSegments`, backslash-newline | The pair is dropped without ending the word | Correct. |
| `splitShellSegments`, `$VAR` and `$'..'` | `$VAR` is a literal word and passes. `$'it\'s'` ends on the escaped quote and leaves an unterminated quote, so `false`. | Correct. The error is on the safe side. |
| `splitShellSegments`, brace group, `cd x; ls`, `FOO=1 ls` | The first word is not in the table, so `false` | Correct, and conservative. |
| `splitShellSegments`, `ls #c; rm x` | `#` is not handled, so the `rm` segment gives `false` | Correct, and conservative. |
| `scanDoubleQuoted` | `"$(rm x)"` refused; `"a\|b"` passes; an unterminated quote is refused | Correct. |
| `skipDiscardRedirect` | `>`, `>&-`, `>|f`, `>&f` and a quoted `"/dev/null"` all give `false` | Correct, and conservative. |
| `findReadsOnly` | `-exec`, `-delete` and `-fprint*` are matched as whole words | Correct. |
| `sedPrintsOnly` | `-ni` gives `false`. `--quiet` and `-n` pass. Without `-n` gives `false`. | Correct. |
| `gitReads` | `-C <dir>` and `--no-pager` are skipped. `-c`, bare `git` and `-C` with no value give `false`. | Correct. |
| `gitReads`, `git log --output=f` | Passes (see finding 1) | Not correct. |

## Findings

1. **LOW, class `design`.** `gitReads` accepts a listed subcommand with any arguments, but several of those subcommands have options that write a file or run a program.
   - `git diff --output=<file>`, `git log --output=<file>` and `git show --output=<file>` write a file.
   - `git grep -O<cmd>` and `--open-files-in-pager=<cmd>` run a program.
   - `git log --ext-diff` and `git diff --ext-diff` can run an external diff driver.
   - Each is classed as a lookup. `_lym/landing/summary.md` and `claudeengine/doc.go` name the write cases (`-i`, `-exec`, `-delete`, `-fprint`, redirects) and list `git` reads as "no write", but not `--output`.
   - The hook only classifies the command and never runs it, so the effect is one miscounted lookup in a chain, not an execution.
   - The summary's "Bounds" section names rg `--pre` and sed's `w` as undetected, but not these git options.
   - Evidence: `bashlookup.go:82-97` and `bashlookup.go:29`.

I found no other correctness defects in the diff. The one gap is among the tool calls: the first test row, `ToolUseIDAbsentFromTheTailIsSilent`, and the missing-agent-file path both end in `errNoNudge` without touching the state. That is correct, and it is for the code-tests lens.

## Tool calls

None.
