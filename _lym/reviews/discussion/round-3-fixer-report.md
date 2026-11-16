# Fixer report — Discussion review, round 3

## Changes

- **F1 (BLOCKING), fixed differently in part.**
  D12's call-shape paragraph now says a model call is one top-level call item of the rollout:
  an `exec` cell is a `custom_tool_call`, and the `collaboration` calls the rollouts show are `function_call` items with their own `call_id`, answered by a `function_call_output`.
  The payload sentences hold only for a nested call.
  The nudge lists top-level calls in rollout order, puts a nested call's payload in the latest `exec` cell,
  and gives no nudge to a payload whose `tool_name` is neither `Bash` nor `apply_patch`, leaving the rollout unread, as Claude's matcher leaves an `Agent` call unjudged;
  so a `function_call` is never looked up by a payload id or put in a cell.
  A new bullet says a `function_call` is neither a lookup nor an `apply_patch`-only cell and so ends a `lookup_run` or `edit_run` chain.
  The Facts now list the `functions`, `clock` and `collaboration` tools as the `additional_tools` record holds them,
  the 21 `exec` items and the four `function_call` items, and the sibling rollouts' further calls.
  The D12 acceptance criterion gains the chain-ending case.
  Not made: the spike the fix proposed for when it is unknown whether `PostToolUse` fires for a `collaboration` call.
  The rule as written gives such a payload no nudge whether or not the hook fires, so the answer changes nothing in the design;
  the record states that no spike has run one.
  The spike would also write outside this round's write surface (a hooks config, a Codex session and its rollout).
  The review's "sleep, wait and clock" was corrected against the rollout: `wait` is in `functions`, and `clock` is the namespace holding `sleep`.
- **F2 (MEDIUM), fixed as proposed.**
  D1 defines a launch-spec literal as a `shuttleengine.Spec` composite literal that sets `Prompt`,
  and says a literal that sets none, such as the `awaitLiveJudge` and `awaitLiveSeed` probes, is skipped by D1's scan, D4's scan and the producer table.
  D1 says a producer that fills its prompt inline, as `judgeCall` and `runSeedSpawn` do, gets its prompt render split out in its own package.
  D4's scan names launch-spec literals (D1) and skips an attach probe's literal.
  The D1 and D4 acceptance criteria and the Notes line about the renderer match.
- **F3 (LOW), fixed differently in part.**
  D12's read-again bullet resolves both paths to absolute and counts an earlier read only when its cell's `custom_tool_call` line is not older than the file's last modification.
  The Facts record that the spike's `parsed_cmd` entries name `notes.txt` as written and that the payload carries `cwd`.
  Not made: resolving the sliced read's path against the payload's `cwd`, as the fix proposed.
  A nested `exec_command` may set its own `workdir` (the `exec` tool schema declares `workdir?: string`, and the 2026-11-15 lym rollouts set it),
  while the payload's `cwd` is the session's, so each path is resolved against its own call's `CommandExecution` item's `cwd`,
  and the nudge waits for the sliced read's own item with the bound Claude's judgment uses for its message.
  This came from the self-check's fix-proposal finding.

## Self-check

- fix-consistency: no findings.
- fix-claims, LOW: "Every other model call … is a `function_call` item" claimed more than the rollouts show. Fixed as proposed: scoped to the calls the rollouts show, with any other top-level call item judged as a `function_call`.
- fix-proposal, MEDIUM: resolving the sliced read's path against the payload's `cwd` is wrong when a nested call sets `workdir`. Fixed as proposed: both paths resolve against their own `CommandExecution` item's `cwd`, with a bounded wait for the sliced read's item.
- fix-way-forward: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
