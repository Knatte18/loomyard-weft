## Goal

A Codex session gets the question-tool deny and the batching nudge Claude runs

Takes: codex-standing-instructions' codexengine launch line, whose one hooks.PreToolUse and hooks.PostToolUse settings these entries join
Delivers: the request_user_input deny on an autonomous Codex launch; the Codex batching nudge reading the rollout's top-level calls and exec cells, judged by Claude's rules and keyed by transcript_path; the read-only lookup check moved out of claudeengine into a package both engines import; codex.yaml's batching_nudge block
Exports: the moved lookup check's package under internal/shuttleengine

## Scope

The part changes these packages:

- internal/shuttleengine/codexengine (about 800 changed lines)
- internal/shuttleengine/claudeengine (about 80 changed lines)
- internal/shuttleengine/bashlookup (about 280 changed lines)
- internal/shuttlecli (about 60 changed lines)
- internal/shuttleengines (about 40 changed lines)

## Decisions

### D12 — The hooks Claude runs reach Codex

An autonomous Codex launch denies Codex's question tools through `PreToolUse`, the counterpart of Claude's AskUserQuestion deny.
The deny entry takes the `*` matcher and denies a call whose payload tool name ends in `request_user_input` or in `request_user_input_async`,
so a name the payload prefixes, as it prefixes `spawn_agent`, still matches.
The entry joins the one `hooks.PreToolUse` setting beside the guard rows, since a second `-c` of that key replaces the first.

The batching nudge runs on Codex: a `PostToolUse` entry with the `*` matcher judges the calling chain's recent calls from the Codex rollout.
It hands its line to the model as `additionalContext` with a `systemMessage`, never as a block;
the entry joins the one `hooks.PostToolUse` setting beside the activity and creel mail entries.

A Codex model call is one top-level call item of the rollout.
An `exec` call is one cell: a `custom_tool_call` named `exec` whose JavaScript runs any number of nested `tools.exec_command` and `tools.apply_patch` calls.
The other model calls the rollouts show, the `collaboration` namespace's `spawn_agent` and `send_message`,
are `function_call` items, each with its own `call_id`, answered by a `function_call_output` with that `call_id`, and run no nested call.
Any top-level call item other than an `exec` cell is judged as a `function_call` is below.
`PostToolUse` fires once per nested call and never for the `exec` cell itself.
A nested shell call's payload carries `tool_name` `Bash` and its command as `tool_input.command`;
a nested `apply_patch` call's carries `tool_name` `apply_patch` and the patch text as `tool_input.command`.
A nested call's `tool_use_id` is the id of its `item_completed` item in the rollout, a `CommandExecution` or a `FileChange`.
The nudge reads the rollout at the payload's `transcript_path` and lists its top-level calls in rollout order:
each `exec` cell, holding the nested items that fall between its call and its output, and each `function_call`.
A nested call's payload belongs to the latest `exec` cell, whose output is not yet written.
A payload whose `tool_name` is neither `Bash` nor `apply_patch` gets no nudge, and the nudge leaves the rollout unread for it,
as Claude's matcher leaves an `Agent` call unjudged;
so a `function_call` is never looked up by a payload's id and never put in an `exec` cell.
No spike has run a `collaboration` call under a `PostToolUse` hook, and this rule holds whether the hook fires for one or not.
One top-level call counts as one message does on Claude, and the nudge judges Codex's calls by Claude's rules:

- a lookup is a `Bash` call whose command passes the read-only lookup check Claude's Bash calls get, the check moving to a package under `internal/shuttleengine` that both engines import;
  a cell whose only nested call is one lookup counts toward `lookup_run` as a single-lookup message does;
- a sliced read is a `Bash` call that prints a line range of one file, such as `sed -n` with a range or `head` and `tail` with a count,
  judged against `slice_lines` and the read-again rule as a sliced Read is;
  an earlier read of that file is a `Bash` call of an earlier cell whose `CommandExecution` item's `parsed_cmd` lists a `read` of that path, such as a `cat` of it;
  it counts only when that cell's `custom_tool_call` line is not older than the file's last modification, as Claude's rule states;
  `parsed_cmd` holds the path as the command wrote it, such as `notes.txt`, so both paths are resolved to absolute before they are compared,
  each against its own call's `CommandExecution` item's `cwd`, a `file://` URL in the spike rollout,
  since a nested call may set its own `workdir` and the payload's `cwd` is the session's;
  the nudge waits for the sliced read's own item as Claude's judgment waits for its message, and a call whose item is missing at the bound gets no read-slice nudge;
- an edit is an `apply_patch` call, and an `apply_patch`-only cell, every nested call of which is an `apply_patch` call,
  counts toward `edit_run` as an edit-only message does;
- a `function_call` is neither a lookup nor an `apply_patch`-only cell,
  so it ends a `lookup_run` or `edit_run` chain as a message holding an `Agent` call does on Claude.

A Codex chain's nudge count and `max_nudges` cap are keyed by the payload's `transcript_path`,
since a Codex payload carries no `agent_id` and a child's carries its parent's `session_id`;
so each child has its own count and cap, as Claude's subagents do.
codex.yaml gains its own `batching_nudge` block with claude.yaml's keys and defaults.

Rationale: a Claude session gets both hooks (`internal/shuttleengine/claudeengine/doc.go`), and a Codex session gets neither.
A block after a tool call makes Codex read that call as failed and may run it again (`internal/shuttleengine/codexengine/doc.go`), which a nudge on a successful call must not invite.
Facts (codex-cli 0.164.0): the binary holds the `request_user_input` tool, and `codex features list` shows `default_mode_request_user_input` under development and off;
`PreToolUse` fires for non-shell tools, and its payload names `spawn_agent` as `collaborationspawn_agent` (`docs/research/codex/codex-twill-spike-2026-10-24/report.md`);
`post_tool_use` delivers `additionalContext` with `systemMessage` to the model (`docs/research/creel/creel-hook-channels-2026-10-28/report.md` case 5, `docs/research/creel/creel-midturn-trust-2026-10-28/report.md`).
A 0.164.0 lym run's rollout (`~/.lym/codex/sessions/2026/11/15/rollout-2026-11-15T19-12-44-01a1e069-….jsonl`) shows the call shape:
its `additional_tools` record lists `exec`, `wait`, `request_user_input` and `request_user_input_async` in the `functions` namespace,
`sleep` in the `clock` namespace,
and `spawn_agent`, `send_message`, `followup_task`, `interrupt_agent`, `list_agents` and `wait_agent` in the `collaboration` namespace.
Its 21 `exec` calls are `custom_tool_call` items, and its four `collaboration` calls, two `spawn_agent` and two `send_message`, are top-level `function_call` items;
the run's two sibling rollouts of that day hold further `spawn_agent` and `send_message` calls the same way.
Its nested commands and patches show only as `item_completed` `CommandExecution` and `FileChange` items,
a `CommandExecution` carrying its `cwd` and a `parsed_cmd` list that marks a `cat` or `sed -n` of a file as a `read` of its path.
A `codex exec` spike on 2026-11-16 with `*`-matcher `PreToolUse` and `PostToolUse` hooks logging their payloads
(rollout `~/.codex/sessions/2026/11/16/rollout-2026-11-16T10-25-31-01a1e3ad-….jsonl`) ran four `exec` cells, a `cat`, a `sed -n` range, an `apply_patch` and a two-command shell call:
both hooks fired once per nested call and never for a cell, with `tool_name` `Bash` or `apply_patch`, `tool_input.command` holding the command or the patch text,
`tool_use_id` equal to the rollout item's id, and `transcript_path`, `turn_id` and `cwd` present;
its `parsed_cmd` entries name the file as written, `notes.txt`, for `cat notes.txt` and `sed -n '1,2p' notes.txt`.

### Added after Discussion (arbiter, 2026-11-16): The assist-trial schedule risk has no subject

Decision: Drop the open risk on codex-assist-review-trial and close Q14 as answered: this task takes no scheduling call on that trial. The three runs the trial chose (operator, 2026-11-15), prime-is-root-beam, hub-stop-button and machine-restart-shortcut, no longer exist as tasks: the operator merged them into adopt-build-resplit (2026-11-16) to be split again from scratch, and that entry carries no code_assist_reviewer_model setting, so no run with a Codex assist seat is queued or running. Any runs the trial picks afresh are the trial note's own choice, so this record states no scheduling bound.

Rationale: Q14 asked whether the three chosen runs go before or after this task lands. Board facts: lym board find code_assist_reviewer_model matches only the trial note, and adopt-build-resplit's body says the beam's remaining members were merged into it. So there is nothing to schedule, and choosing future trial runs belongs to codex-assist-review-trial, not to this task's design.

### Added after Discussion (arbiter, 2026-11-16): Round-3 review findings are answered by the record as it stands

Decision: Treat the round-3 findings d12-non-exec-calls-outside-cell-model (F1), d1-scan-reaches-probe-literals (F2) and d12-read-again-path-resolution-unstated (F3) as answered by D1, D4 and D12 as written; the plan carries D12's function_call rules, D1's launch-spec literal definition (a shuttleengine.Spec literal that sets Prompt, attach probes skipped, inline prompts split into a renderer) and D12's per-call cwd resolution as stated, with no further change.

Rationale: The Discussion review reached its hard cap at round 3 with these keys open. D12 now scopes the payload and cell sentences to nested calls, gives a non-Bash, non-apply_patch payload no nudge and leaves the rollout unread for it, and ends a lookup_run or edit_run chain at a function_call; the 2026-11-15 rollout holds 21 custom_tool_call exec items and four collaboration function_call items (two spawn_agent, two send_message) as D12 states. D1 and D4 skip a Spec literal that sets no Prompt; bouncer.go's awaitLiveJudge and awaitLiveSeed literals set only OutputFiles, Role, Segment and Round, while judgeCall and runSeedSpawn set Prompt inline. D12 resolves both read paths against each call's CommandExecution cwd; the 2026-11-16 spike rollout carries that cwd as a file:// URL and parsed_cmd paths as written (notes.txt).

## Constraints

- `PATTERN-shuttle-provider-seam`: Claude's user-setup resolution, session scope and file locations stay in `claudeengine`, and reach `codexengine` only through `shuttleengine` interfaces wired in `internal/shuttleengines`; the rollout reading for the batching nudge stays in `codexengine`, and the read-only lookup check both engines' nudges use sits in its own package under `internal/shuttleengine`.
- `PATTERN-stencil-ownership`: engine text in engine blocks, resolved by `internal/stencilstore`'s one resolver; skills stay typed on the launch spec, and the skill listing is no request to load one.
  Its background's edit-directive section changes with D3: the engine copies are the directive's only statement.
- `PATTERN-heddle-instructions`: the shipped-entry changes and the new target live in `internal/heddleengine`; `HEDDLE.md` states the new target, and the background names the hub reconcile as a file-target render site.
- `PATTERN-child-processes`: the plugin hook runs are logged, waited on and registered as the pattern says.
- `PATTERN-tolerant-records`: the manifest record is written through `internal/state` and read tolerantly.
- `PATTERN-config-layers`: the new codex.yaml keys ship through `configengine` with their template defaults.
- `PATTERN-cli-cobra`: the verb joins shuttle's cobra subtree.
- `PATTERN-test-speed`: the parity test and the scans run at `tier0` on fixtures, and no test pins a stencil's prose.

## Auto-mode assumptions

- D1: no exemption table; every compared difference found has a Codex channel.
- D6: subagents run on the session's model; 16 sits above the widest fan any stencil asks for.
- D12: the nudge keeps separate settings per engine rather than moving to a shared module, which would need a config reinterpretation.

## Open risks

## Acceptance criteria

- An autonomous Codex launch denies a call whose payload tool name ends in `request_user_input` or in `request_user_input_async`,
  through its entry in the one `hooks.PreToolUse` setting beside the guard rows (D12).
- Under codex.yaml's `batching_nudge` thresholds, a Codex chain of `lookup_run` single-lookup `exec` cells,
  a `Bash` line-range read of a file under `slice_lines` lines or of one an earlier `Bash` call read and unchanged since, its path given relative,
  or `edit_run` `apply_patch`-only `exec` cells gets the nudge line as additional context, without the call being marked failed;
  a top-level `function_call` such as `spawn_agent` ends a `lookup_run` or `edit_run` chain;
  each chain's count and `max_nudges` cap are its own, keyed by `transcript_path` (D12).
