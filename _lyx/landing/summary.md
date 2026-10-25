# Report: creel mail delivery through a tool-boundary hook

Adds `docs/research/creel-hook-delivery-2026-10-25/report.md` and two raw run logs beside it; no production code, config or plugin changes.

## What was tested

A flag file per strand, set when the strand has unread mail, and a `PostToolUse` hook that tests the flag first and, when it stands, claims the mail by atomic rename and hands it over as hook context.
Creel's side was played by scratch scripts, not the daemon.
The prototype stayed under the worktree's `.scratch/` and no hook was installed in the operator's or the plugins' settings.

## Results

- Claude, mid-turn: every message posted during one long turn reached the receiver at the next tool boundary, within 25 ms of the tool call's end, with the 40 s call uninterrupted.
- Claude, idle: the pane path delivered 19 ms after post.
- Race: 146 messages posted across turn ends with both paths live; 130 went by hook, 16 by pane, each claimed exactly once.
- Hook cost with no mail: about 0.6 ms over a bare process start, no tokens; with mail about 3 ms for one message.
- SendMessage reaches the receiver at the same boundary; one sample each cannot rank their delay.
- The receiver read the mail as a strand message and refused an operator impersonation; it answered by `lyx creel post --reply` only where the strand's prompt told it to.
- Codex was not tested live: the account hit its usage limit until 16:45 local.
  The 0.164.0 binary lists `post_tool_use` and `pre_model_request` as hook events and an `additional_context_limit` setting; the report gives the steps to finish the run.

## Design decisions

- Exactly-once was tested for the file protocol only, as the board entry's scope allows no change to `internal/creelengine`; the report names this as a limit.
- The Codex result is open, not a negative: the report recommends deciding after a live run and says how.
- The recommendation keeps creel's idle-pane delivery and flags that dropping the reminder removes the guard against one agent's text posing as an instruction, resting on one probe.
