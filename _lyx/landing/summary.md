# Webster: recover from audit and verify-gate halts without the operator

A Webster run that halts on its run-exit audit or its verify gate can now be recovered by the agent that receives the refusal, without discarding the run's batches.
This closes four failures from the `board-model` run: the gate waiting forever on a hung background shell, a fork-contract-write finding that `accept-audit` could never accept, garbled way-forward text, and recoveries that name a command the agents are denied.

Closes #362
Closes #365

## What changes

- **Shuttle bounds the wait on background shells.**
  `claudeengine` reports outstanding background tasks (kind `fork` or `shell`, id, label) on each waiting event, from the Stop payload's `background_tasks[]` and from the transcript's unmatched launches.
  A turn end whose outstanding tasks are all shells counts as a turn end after the new `shuttle.yaml` key `background_shell_wait_min` (default 10; non-positive is a config error).
  The check runs on every poll tick, an expired shell stays expired for the run, and any outstanding fork keeps the wait as before.
  A `RunSpec` can declare awaited shell command prefixes; Webster's Master declares `lyx webster recover-batch`, which `recovery_timeout_min` already bounds.
- **Expired shells are surfaced.**
  `lyx webster run` adds an envelope warning per expired shell, a `## Background shells waited out` section in `summary.md` and a Go-authored friction note.
- **Agents' Bash stdin defaults to `/dev/null`.**
  Shuttle's PreToolUse(Bash) hook rewrites each command to start with `exec </dev/null; `, which closes the `python3 -` hung-stdin class at its source.
  Heredocs, pipes and redirects still feed the command they are attached to.
- **Stencils.**
  The fixer and batch fork stencils name `outcome.yaml` and `summary.md` as files the fork never writes; Master's template says it reads and writes through its tools, never a script or interpreter.
- **A fork-contract-write finding is checkable.**
  It clears when the contract file is absent, or when Master's transcript records a successful later Write or Edit of it.
  `accept-audit`, `recover-batch`, `--fresh` and the pending-findings check share this rule.
  Other paths under `_lyx`, `.lyx` and scratch stay unverifiable.
- **New verb `lyx webster reset --to start|pre-fix`.**
  It runs through a new gated primitive in `internal/fabricengine/destroy.go`.
  `start` resets to the run's start commit, or the octopus merge-base of diverging batch starts; `pre-fix` resets to the verify gate's pre-fix head, now persisted in `state.json`.
- **Way-forward text.**
  Every Webster refusal, refusal-spec row and stencil that named a git reset now names `lyx webster reset`.
  The run-exit audit and pending-findings messages state each finding once, give a reason per uncheckable path, and end in a numbered command list.
- **`PATTERN-no-denied-recovery`** with its scan test: no refusal, stencil, spec or ly skill offers a command the agents are denied.

## Design decisions to review

- **Reset guard.**
  `reset` can only discard commits above a run-recorded commit on the task's own pair branch, plus uncommitted tracked changes to paths the run wrote.
  It refuses on the parent branch, detached HEAD, a non-pair branch, a held run lock, no recorded target, a missing or non-ancestor target, a merge in progress, and any other dirty tracked path; there is no `--force`.
  The own-path set comes from transcript Write/Edit results, by path; a path the run wrote and someone else edited later is still discarded, and a change made through Bash makes the verb refuse (safe direction).
- **Absent contract files clear the audit finding only.**
  The run stays in flight, and the way forward names the step that re-enters Master to write them.
- **Pre-fix head lifetime.**
  Written at the first failed gate evaluation of each shuttle run, overwritten by the next run's, and cleared by a passing gate, a successful `reset` and `run --fresh`.
- **Fabric's manual weft reset is out of scope.**
  It appears only in Go comments, in no refusal, stencil or spec.
- **Not done:** no `python` deny, and the global deny on `git reset --hard` stays.

## To check by hand

- The stdin rewrite relies on Claude Code honouring `updatedInput` from a PreToolUse hook in the mode lyx agents run in, and on allow/deny rules matching the original command; confirm in a live agent session that an allowed and a denied Bash command behave as before.
- Run the `board-model` shape end to end: a hung background shell ends the wait after the bound, and a fork-contract-write halt recovers through `accept-audit` or `lyx webster reset`.
- The integration tier (`reset` against hubforge fixtures) and the sandbox suite were not run for this description.
