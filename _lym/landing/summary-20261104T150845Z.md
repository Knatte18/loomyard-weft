# The batching hook counts read-only Bash lookups and reaches forks

The turn-savings measure (`docs/research/turn-savings-measure-2026-11-04/report.md`) found the batching nudge firing 4 times in 24 runs.
It had two blind spots: lym sessions search with Bash `grep`, which the lookup chain never saw, and the hook skipped every call from a subagent or fork, where most edit chains and lookup chains happen.
This change closes both gaps and leaves every `batching_nudge` threshold unchanged.

## What changes

**Bash lookups count in the lookup chain.**
The hook's PostToolUse matcher now includes `Bash`.
A Bash call whose command is a read-only lookup counts as one lookup in the single-lookup chain, as a Read, Grep or Glob call does.
A Bash call with any other command breaks the chain, as before.
`bashlookup.go` makes this call: a quote-aware scan splits the command into segments, and a table keyed by command name checks each segment's arguments:

- `grep`, `rg`, `ls`, `cat`, `head`, `tail` and `wc` with any arguments;
- `find` without `-exec`, `-execdir`, `-ok`, `-okdir`, `-delete` or an `-fprint`/`-fls` action;
- `sed` with `-n` (or `--quiet`/`--silent`) and no in-place flag;
- `git`, past `-C <dir>` and `--no-pager`, running `log`, `show`, `diff`, `status`, `blame`, `grep`, `ls-files`, `ls-tree`, `rev-parse` or `cat-file`.

The following never count as a lookup: a command substitution (including one inside double quotes), process substitution, subshell, heredoc, background `&`, any output redirect except to `/dev/null` or a descriptor (`2>&1`), an unterminated quote, and an empty command.
A Bash call that is not a lookup returns before the state file's lock and the transcript read, because it cannot complete any breach.
Only a lookup Bash call pays for the transcript read.

**Subagents and forks are judged on their own.**
A call that carries `agent_id` reads that agent's own transcript, `<session>/subagents/agent-<id>.jsonl` beside the session's transcript.
When `transcript_path` already names that file, it is read as is.
The tail's messages and its context reading come from the agent's sidechain lines, so `context_tokens` is measured against the agent's own context.
The state file keeps a separate nudge count and last-nudged message for each agent id, under a new `agents` map.
`max_nudges` caps each agent separately from the main session.
An agent id that holds a path separator, or is `.` or `..`, gets no nudge.
`latestReading` and `tailMessages` take a `sidechain` switch for this; the session-usage caller passes `false` and is unchanged.

**The re-measure keeps working.**
The nudge lines are unchanged, so `events.jq`'s breach matching still classifies each firing.
`events.sh` already reads fork transcripts, which is where a fork's nudge attachment is recorded.
`breaches.sh`'s comment now says that its tool reading is the hook's pre-change lookup set, and that the class reading is the closer match to the hook's lookups now.
`run-all.sh` and its table columns are unchanged.

## Design decisions

- `&&`, `||` and a newline join segments the same way a pipe and `;` do: a chain made only of lookups is still a lookup.
  The entry names only pipes and `;`-chains, but nothing a `&&`-chain adds makes it a write.
- The command set is the entry's list, plus the git reads `blame`, `grep`, `ls-files`, `ls-tree`, `rev-parse` and `cat-file`, which read as `log` and `show` do.
  A git global option other than `-C` and `--no-pager` disqualifies the call, since `-c` can set a program git runs.
  Pipe filters such as `sort` or `xargs` are not in the set, so `grep … | sort` breaks a chain.
  Adding one is a row in `lookupCommands`.
- The agent transcript path is derived from Claude Code's on-disk layout: the same `subagents` directory that `forkActivity`, `SessionUsage`, the fork audit and `locateSessions` read.
  That directory name and the `agent-` file prefix are now named constants (`subagentsDirName`, `subagentTranscriptPrefix`) in `transcriptread.go`, and the existing callers use them.
- The main session's count stays at the state file's top level (`nudges`, `nudged_message`), so a state file seeded by an older binary still reads.

## Bounds

- The scan reads static text: an argument that makes a listed command run a program, such as rg's `--pre` or a sed script's `w` command, is not detected.
- The hook now starts one `lym` process after every Bash call.
  A non-lookup call ends at the input parse.
