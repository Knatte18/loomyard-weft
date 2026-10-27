# Decision record — creel-hook-delivery

## Goal

Creel delivers mail to a working Claude strand at its next tool boundary through a `PostToolUse` hook, so a message reaches a strand in a long turn without ending the turn or losing a tool call in flight (issue #871).

## Scope

- `internal/creelengine`: a per-strand flag file kept while the strand holds unread mail, a store claim for the hook path, a daemon verb serving it, and a single-owner claim shared with the idle pane path's notice.
- `internal/creelcli`: a hidden `lym creel` verb the hook runs, which claims the caller's mail and prints it as the provider's tool-boundary context.
- `internal/hubgeom`: the flag directory in creel's geometry.
- `internal/lymdirs`, `internal/agentname`: the flag directory's name and the flag file name of a strand.
- `internal/shuttleengine`, `internal/shuttleengines`, `internal/shuttleengine/claudeengine`: an optional engine capability rendering tool-boundary context, the registry route to it, and the `PostToolUse` hook in every hub Claude strand's settings.
- Docs: the `doc.go` of each package above that changes behaviour.
- Out of scope: Codex strands, the idle pane path's own rules (reminder for agent mail and long messages), moving agent messaging off SendMessage.

## Decisions

### Flag file per strand with unread mail

Decision: creel keeps one empty file per strand that holds unread mail, at `<hub>/_board/.lym/creel/flags/<flag name>`, created when the strand's first unread message arrives and removed when none is left.
The flag name is the full strand name with every `:` turned into `.`, which no name segment holds, so it is a valid file name on every OS and maps back one to one.
The store recomputes the set after every change it journals and creates or removes only the files that differ; the daemon reconciles the whole directory at start.
Programs get no flag.
Rationale: the hook's no-mail path stays one file test with no process start; the store already serialises every pool change, so the flags follow the pool without a second writer.

### Hook rendering in claudeengine

Decision: claudeengine renders a `PostToolUse` hook with no matcher into a Claude strand's settings whenever its engine is told a hub board directory.
The hook is plain POSIX `sh`: it forms the flag name from `$LYM_STRAND_NAME` with shell parameter expansion alone, exits 0 at once when that flag file is absent, and otherwise changes to `$CLAUDE_PROJECT_DIR` and runs `lym creel hook-read claude`.
A standalone run, told no board, gets no hook, since creel needs a hub.
Rationale: no fork on the common path; the flag directory derives from the board directory the engine is already told.

### Provider-neutral claim, provider-specific output

Decision: a new optional capability `shuttleengine.ToolBoundaryContextRenderer` renders text as the provider's tool-boundary hook output; Claude implements it as `hookSpecificOutput.additionalContext` for `PostToolUse`.
`shuttleengines` hands it out by engine name, and the hidden `lym creel hook-read <engine>` verb uses it.
Rationale: keeps Claude's JSON shape under `internal/shuttleengine/claudeengine` (`PATTERN-shuttle-provider-seam`).

### What the hook claims

Decision: the hook-read verb claims the caller strand's unread, unreserved messages oldest first, agent mail included, and prints one `[creel] #<id> from <sender>: <body>` line per message, the same line the idle path types for a notice.
Under a hold it claims only the hold owner's mail, and while a pause covers the strand it claims nothing.
One claim hands out lines up to an 8 KiB budget, and always at least one message; what is left stays unread with its flag standing for the next boundary.
The claim uses the read reservation and acknowledgement: the messages are journaled read only after the verb wrote its output.
Bound: the hook skips the reminder's guard for agent mail, which was there so an agent's text could not pose as an instruction; the line still opens with creel's stamped sender, and only mail the strand already holds is handed over.
The budget bounds one boundary's context; a single message over the budget is handed over whole.

### One claim for both paths

Decision: the idle pane path reserves a notice in the store before it types it and releases the reservation when the send was busy, so the hook and the pane path claim through the same reservation and each message is delivered once.
Rationale: today the pane path types an unreserved notice and journals the read afterwards, so a hook read in between would hand it out twice.

### Hook verb output on failure

Decision: `hook-read` writes nothing to standard output on a refusal or a daemon error, writes the error to standard error and exits 1, which Claude Code shows as a non-blocking hook error; its call timeout is 10 seconds.
Rationale: standard output of a `PostToolUse` hook is parsed as hook JSON, so the usual JSON error envelope would be misread.

## Constraints

- `PATTERN-shuttle-provider-seam`: Claude's hook JSON stays in claudeengine; only `internal/shuttleengines` reaches the concrete engine.
- `PATTERN-agent-name`: the flag name is formed in `internal/agentname`.
- `PATTERN-durable-vs-ephemeral-state`: the flag directory's name is a `lymdirs` constant under the board-level `.lym`.
- `PATTERN-pane-text-injection`: the hook types nothing into a pane; the pane path's typing is unchanged.
- `PATTERN-tolerant-records`: no new journal record kind; flags are derived state, never journaled.
- Creel's daemon stays the only writer of the journal and of the flags.

## Auto-mode assumptions

- Claude Code passes the session's environment, `LYM_STRAND_NAME` and the pane's `PATH` included, to hook processes, and sets `CLAUDE_PROJECT_DIR` to the session's project directory.
- The hook process runs inside the strand's pane process tree, so creel's peer-credential identity resolves it to the strand.
- An 8 KiB context per boundary is within what Claude Code passes to the model unshortened.

## Open risks

- The injection guard for agent mail is weaker on the hook path; the research report rests on one probe.
- A flag left standing while no daemon runs makes the hook start `lym`, which starts the daemon and reconciles the flags; until then each tool call pays that process start.
- The flag stands while a hold mutes non-owner mail, so the hook starts `lym` at each tool call of a held strand until the hold ends.

## Acceptance criteria

- A post to a strand creates its flag file; reading, forwarding, withdrawing or returning its last unread message removes it; a program receiver gets none; a daemon start reconciles a stale or missing flag.
- The hook-read claim hands out the strand's unread mail oldest first as composed lines, marks it read on acknowledgement, honours the hold and the pause, and stops at the budget with at least one message.
- A notice the pane path is typing is not handed out by a concurrent claim, and a notice claimed by the hook is not typed.
- The `hook-read` verb prints the Claude `PostToolUse` additional-context JSON with one line per message, prints nothing when nothing is unread, and refuses to standard error.
- A hub Claude strand's settings carry the `PostToolUse` hook; a standalone one does not; under `sh` the hook exits 0 without running `lym` when the flag is absent or `LYM_STRAND_NAME` is empty, and runs it when the flag stands.

## Notes for the plan writer

- `Store.do` is the one place every journaled change passes; the flag sync belongs at its end, under the mutex.
- `composeNotice` already renders the line the hook prints.
- `hookcmd_integration_test.go` in claudeengine runs rendered hook commands under `sh`.
