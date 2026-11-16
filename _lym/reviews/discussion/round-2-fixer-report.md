# Round 2 fixer report — discussion

Every finding's premise held against the code and the rollouts; all eleven are fixed.

## Changes

- F1 (fixed as proposed, research done):
  read the lym run rollout `~/.lym/codex/sessions/2026/11/15/rollout-2026-11-15T19-12-44-01a1e069-….jsonl`, and ran a `codex exec` spike on 2026-11-16
  with `*`-matcher `PreToolUse` and `PostToolUse` hooks logging their payloads (rollout `~/.codex/sessions/2026/11/16/rollout-2026-11-16T10-25-31-01a1e3ad-….jsonl`).
  Result: each model call is an `exec` cell, and both hooks fire once per nested call, never for the cell.
  A shell call's payload has `tool_name` `Bash` and an `apply_patch` call's has `tool_name` `apply_patch`, each with `tool_input.command`;
  `tool_use_id` equals the rollout item id, and `transcript_path` is present.
  D12 now states that shape and counts one `exec` cell as one message.
  It says how single-lookup and `apply_patch`-only cells are told apart, and names an earlier read as a `Bash` call that `parsed_cmd` lists as a `read` of the path, such as a `cat`.
  The Rationale records both sources, and the last acceptance criterion is restated in cells.
- F2 (fixed as proposed): the deny matches `request_user_input` or `request_user_input_async`.
  The Rationale sentence is replaced with the `additional_tools` fact, and the criterion names both tools.
- F3 (fixed as proposed): D1's producer table and scan, and the first criterion, are keyed by the enclosing function of each production launch-spec literal.
- F4 (fixed as proposed): D1 writes a lasting copy of the record into a `manifests` directory at the run-directory root.
  Cleanup, the orphan sweep and the run lookup by strand all leave that directory alone.
  D11's report opens from that copy, and the third criterion checks the copy survives cleanup.
- F5 (fixed as proposed): D4 adds a `shuttleengine` call returning the engine a model choice resolves to, the set's default for an empty choice;
  every producer passes it to its renderer and spec; a D4 criterion covers it.
- F6 (fixed as proposed): D1 compares items from `agents-file` and `agents-prompt` under the shared target's key and leaves only single-engine entries uncompared;
  the second criterion carries the case.
- F7 (fixed as proposed): Open risks reads `None.`, and Q14 is resolved with a pointer to the `Added after Discussion` entry of 2026-11-16.
- F8 (fixed as proposed): D12 places the deny entry in the one `hooks.PreToolUse` setting beside the guard rows; the criterion says so.
- F9 (fixed as proposed): D12 keys a Codex chain's count and `max_nudges` cap by `transcript_path`; the criterion says so.
- F10 (fixed as proposed, then narrowed by the self-check):
  each `codex-file` render records the worktree-relative paths of the in-worktree imports it resolved, and heddle's matcher adds them to the fixed re-render list.
  A bound covers imports outside the worktree, and a D9 criterion was added.
- F11 (fixed as proposed): D10's bound says its size check leaves out the plugin session-start contexts, so a passed seat can still be refused at launch.

## Self-check

- fix-claims, fix-proposal, fix-way-forward (BLOCKING, merged): the orphan sweep deletes a `manifests` directory, since it holds no `run.json`, and the strand lookup counts it as a damaged run.
  Disposition: fixed as proposed, D1 says the sweep and the strand lookup skip that directory.
- fix-consistency (MEDIUM): D10's write bound contradicts D1's lasting copy.
  Disposition: fixed as proposed, D10 and its criterion say the verb's `Prepare` keeps no lasting copy.
- fix-claims (MEDIUM): Claude's `lookup_run` counts single-lookup messages (`claudeengine/batchingnudge.go:37`, `:470`), not lookup-only ones.
  Disposition: fixed as proposed, D12 and its criterion count cells whose only nested call is one lookup.
- fix-proposal (MEDIUM), fix-way-forward (LOW), merged: D9's import triggers had no mechanism in the static `ReRender` table, and an import outside the worktree cannot be a trigger.
  Disposition: fixed as proposed, D9 records resolved in-worktree imports per render for the matcher and bounds out-of-worktree imports.
- fix-consistency (LOW): support log Turn 3 still kept the assist-trial risk open.
  Disposition: fixed as proposed, the entry now points at its later drop.
- fix-claims (LOW): "as Claude's subagents do" was not checked for the cap; the lens raised it as no defect.
  Disposition: no change, the count is per chain at `batchingnudge.go:166` and the lens withdrew it.

## Disputed

None.

## Deferred

None.

## Rulings

None.
