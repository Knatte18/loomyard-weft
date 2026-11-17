# Codex gets the question-tool deny and the batching nudge

A Codex session now gets the two hooks a Claude session already runs, through the one `hooks.PreToolUse` and `hooks.PostToolUse` settings of the Codex launch line.

## What changes

- An autonomous Codex launch denies a call whose payload tool name ends in `request_user_input` or `request_user_input_async`.
  The entry takes the `*` matcher and matches the suffix, since Codex prefixes a namespaced tool name, as it does for `spawn_agent`.
  The deny steers the model to state its question as its final message and end the turn, and the developer-instructions deny notice announces it.
- A `*`-matcher `PostToolUse` hook runs `lym shuttle hook-batching codex` and nudges a Codex thread by Claude's batching rules.
  The line reaches the model as `additionalContext` with a `systemMessage`, never as a block,
  because a block after a tool call makes Codex read the call as failed and may run it again.
- `codex.yaml` gains a `batching_nudge` block with `claude.yaml`'s keys and defaults.
  The values are validated at load while the hook is on.
- The read-only lookup check moves out of `claudeengine` into the new package `internal/shuttleengine/bashlookup`, which both engines import.
  The transcript-tail read and the line count move beside it as `shuttleengine.ReadFileTail` and `shuttleengine.CountLinesUpTo`.

## Design

- The nudge reads the rollout at the payload's `transcript_path` and lists its top-level calls in order:
  each `exec` cell with the nested calls that fall between its call and its output, and each `function_call` such as `spawn_agent`.
  `PostToolUse` fires per nested call and never for the cell, so a nested call belongs to the latest cell whose output is unwritten.
- One top-level call counts as one message does on Claude:
  - A cell whose only nested call is one read-only lookup counts toward `lookup_run`.
  - A cell of `apply_patch` calls only counts toward `edit_run`.
  - A `function_call` is neither, so it ends either chain.
  - A payload whose tool is neither `Bash` nor `apply_patch` gets no nudge, and the rollout is left unread for it.
- A `Bash` line-range read (`sed -n` with a range, `head` or `tail` with a count) is judged against `slice_lines` and the read-again rule.
  An earlier read is a `read` in a prior cell's `parsed_cmd` that is not older than the file's last modification.
  Both paths are resolved to absolute against their own call's `cwd`, since a nested call may set its own `workdir`.
  A call whose rollout item is not yet written at the wait bound gets no read-slice nudge.
- The nudge count and the `max_nudges` cap are keyed by `transcript_path`, because a Codex payload carries no `agent_id` and a child's carries its parent's `session_id`.
  Each child therefore has its own count and cap, as Claude's subagents do.
- Settings stay per engine rather than moving to a shared module, which would need a config reinterpretation.
- The rollout reading stays in `codexengine`, and the lookup check sits in its own package, as `PATTERN-shuttle-provider-seam` requires.
  The pattern and `docs/overview.md` carry the change.
- The call shapes come from a Codex 0.164.0 rollout and a `codex exec` spike with logging hooks.
  They showed `tool_use_id` equal to the rollout item's id and a `parsed_cmd` naming a file as written.
  No spike ran a `collaboration` call under a `PostToolUse` hook, so the `function_call` handling rests on the rollout shape alone and holds whether the hook fires for one or not.

## To check by hand

No regression check or integration run covers a live Codex session.
Run an autonomous Codex launch and check that a `request_user_input` call is denied.
Check that a chain of single-lookup cells, a line-range read and a chain of `apply_patch` cells each produce the nudge line without the call showing as failed.
