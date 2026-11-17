# Codex launches deny the question tools and run the batching nudge

A Codex session now gets the two hooks a Claude session already had:
the deny of the operator-question tools on an autonomous launch,
and the batching nudge that adds one line of context when the session's recent calls break lym's batching rules.

## Shared lookup check

The read-only lookup check moves out of `claudeengine` into a new package, `internal/shuttleengine/bashlookup`, which both engines import.
`IsReadOnlyLookup` is the moved check, unchanged.
`LineRangeFile` is new: it names the one file a command prints a line range of (`sed -n 'A,Bp' f`, `head`/`tail` with a line count).
The bounded tail read and the capped line count the Claude nudge used move to `shuttleengine` as `ReadFileTail` and `CountLinesUpTo`, so the Codex nudge reuses them.

`internal/shuttleengine`'s provider-seam scan treated every package under `internal/shuttleengine/` as a concrete engine.
It now has a `sharedPackages` list naming `bashlookup`, so an engine may import it.
`PATTERN-shuttle-provider-seam`'s background states the exemption.

## Question-tool deny

An autonomous launch (`!spec.Interactive`) puts a `*`-matcher entry first in its one `hooks.PreToolUse` setting, before the guard rows.
The entry denies a payload whose `tool_name` ends in `request_user_input` or `request_user_input_async`, so a namespace-prefixed name such as `functionsrequest_user_input` still matches.
Its notice opens the developer instructions' deny notice.
`toolHooks` groups the launch's tool hooks, the guards, the deny and the batching state file, as `hookSettings`' input.

## Codex batching nudge

`codex.yaml` gains a `batching_nudge` block with `claude.yaml`'s keys and defaults.
`LoadConfig` refuses a threshold below its least value while the block is on, the same way it refuses codex.yaml's other invalid values.
With the block on, Prepare seeds `batching-nudge.json` in the run directory,
and the `hooks.PostToolUse` setting runs `lym shuttle hook-batching codex <state>` on every tool, after the activity hook and before creel's mail hook.
The hook prints `{}` when the verb prints nothing.
The existing `hook-batching` verb needed no change: Codex now implements `BatchingNudger`, so the registry resolves it.

The judgment (`codexengine/batchingnudge.go`) reads a 4 MiB tail of the rollout at the payload's `transcript_path` and lists its top-level calls.
Each `exec` `custom_tool_call` is a cell holding the `CommandExecution` and `FileChange` items that complete before its output.
Every other call, such as a `collaboration` `function_call`, is a top-level call with no nested calls.
The payload's call belongs to the latest exec cell, and is added to it from the payload when its item is not in the rollout yet.
A cell counts as one message does on Claude:

- a cell whose one nested call is a read-only lookup extends a `lookup_run` chain;
- a cell whose nested calls are all `apply_patch` extends an `edit_run` chain;
- any other top-level call, a `function_call` included, ends both chains;
- a `Bash` line-range read fires when the file is under `slice_lines` lines,
  or when an earlier cell's `parsed_cmd` read the same file and the file has not changed since that cell started.
  Each path is resolved against its own call's `CommandExecution` `cwd` (a `file://` URL).

Only a line-range read waits for its own item, polling a bounded number of times;
if the item is still missing at the bound, the read gets no read-slice nudge.
A payload that is neither `Bash` nor `apply_patch`, and a `Bash` call that is no lookup, return before reading the state file or the rollout.
The context threshold comes from the tail's newest `token_count`.
Counts and the `max_nudges` cap are kept per thread, keyed by `transcript_path`, because a Codex payload carries no agent id and a child's carries its parent's session id.
Each count records the cell it nudged, so the nested calls of one cell nudge at most once.
The output carries the line as `additionalContext` with a `systemMessage`, never a block, since Codex would read a block as a failed call.

`rollout.go`'s event parsing is split so the nudge parses the tail with the same code `ContextTokens` uses (`parseRolloutEvents`, `latestContext`).

## Tests

- `bashlookup_test.go` carries the moved lookup table plus `TestLineRangeFile`.
- `filetail_test.go` carries the moved tail-read test plus `TestCountLinesUpTo`.
- `codexengine/batchingnudge_test.go` drives `BatchingNudge` over rollout fixtures, covering:
  - each breach;
  - each way the nudge is held back;
  - per-thread counting;
  - path resolution;
  - the item wait.
- `TestQuestionDenyCommand_DeniesAQuestionTool` (tier 1) runs the deny under `sh`.
- `TestPrepare_QuestionDenyAndBatchingHook` pins that the deny follows `Interactive` and the hook and state file follow `enabled`.
- `TestHookSettings`, `TestLoadConfig_*` and `TestBatchingNudger` gain the new rows.
