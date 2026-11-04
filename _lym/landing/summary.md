# Batching hook counts read-only Bash lookups and reaches forks

The batching nudge hook (`internal/shuttleengine/claudeengine/batchingnudge.go`) almost never fired in the turn-savings measure (`docs/research/turn-savings-measure-2026-11-04/report.md`).
It had two blind spots, and this change closes both.

- The lookup chain matched only Read, Grep and Glob calls, while lym sessions search with Bash `grep`.
  A Bash call whose command is a read-only lookup now counts as a lookup in the lookup-chain breach.
  Any other Bash call breaks a chain, as before.
- The hook skipped every call from a subagent or fork.
  A call carrying `agent_id` is now judged against that agent's own transcript, context reading and nudge count, under the same `batching_nudge` thresholds.

Thresholds are unchanged.

## Design decisions

- `bashlookup.go` classes a command with a quote-aware scan that splits it into pipe, `;`, `&&`, `||` and newline segments.
  A table keyed by command name judges each segment: `grep`, `rg`, `ls`, `cat`, `head`, `tail`, `wc`, `find`, `sed` and `git`.
  - `find` with an action that runs or writes (`-exec`, `-delete`, `-fprint` and the like) is no lookup.
  - `sed` is a lookup only with `-n` and without `-i`.
  - `git` is a lookup only for a read subcommand, past the global options `-C <dir>` and `--no-pager`.
    A git option that writes a file or runs a program (`--output`, `--ext-diff`, `-O`, and abbreviations of four or more characters) is no lookup.
  - A command substitution, process substitution, subshell, heredoc, background `&`, an output redirect to any file but `/dev/null`, an unterminated quote and an empty command are no lookup.
  - The scan reads static text and is a heuristic.
    An argument a listed command treats as a program, such as `rg --pre`, still passes.
- A Bash call that is no lookup returns before the transcript is read, since it makes a message that is neither a lookup nor an edit chain.
  The hook matcher now includes Bash.
- A subagent's or fork's transcript is `<session>/subagents/agent-<agent_id>.jsonl` beside the session's transcript, unless `transcript_path` already names that file.
  An `agent_id` that is not a plain file name part gets no nudge.
  The sidechain flag now selects which chain's lines `latestReading` and `tailMessages` read: sidechain lines for an agent, main-chain lines otherwise.
- The nudge state file keeps the main session's count where it was and adds an `agents` map of counts keyed by agent id.
  `max_nudges` and `context_tokens` therefore apply per calling chain, and the docs, `template.yaml` and the `hook-batching` help say so.
- `docs/research/turn-savings-measure-2026-11-04/breaches.sh` changes only in its comments, which name the class reading as the nearer match to the hook's lookup now.
  `run-all.sh` is untouched and keeps working on runs that land after this change.

## To check by hand

- `BatchingNudge` returns early for a Bash call whose command is no read-only lookup (`batchingnudge.go`, `class design, severity MEDIUM`, bash-early-return-unpinned).
  Check that its tests pin the early return, including that the transcript is not read.
- Run `run-all.sh` on a few runs that landed after this change and confirm it still produces the report's figures.
  This was not run.
- The agent-transcript path layout (`subagents/agent-<id>.jsonl`) is a Claude Code internal.
  Confirm it against a live fork run.
