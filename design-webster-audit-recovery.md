# Webster audit and verify-gate halts are recoverable by the agent

`webster-audit-recovery` · Task · bug, enhancement, webster, shuttle · running · Webster-Burler

# Goal

A Webster run that halts on its run-exit audit or verify gate can always be recovered by the agent that receives the refusal, without the operator and without throwing away the run's batches.
These four notes stopped the `board-model` run on 2026-10-04.

# Decision (operator, 2026-10-05)

The global deny on `git reset --hard` stays.
Every reset a lyx refusal or stencil names is performed by lyx itself, in Go, guarded to the run's own task branch and a commit the run recorded (its start commit or a batch head), refusing on the parent's branch and on uncommitted changes that are not the run's own.
No refusal, stencil or spec (`contracts/specs/refusal-spec.md` included) may name a command the agents are denied.


# Decisions from the task discussion (operator, 2026-10-05)

- **Hung background shell.** The gate waits on a running background shell only up to a configurable bound; past it the turn counts as ended and the driver gets a message naming the shell.
  Subagent forks are still waited on as before.
  The deeper fix is that an agent never starts such a shell at all: find why Master ran `python3 -` reading stdin, and close that at its source (stencil or tooling).
- **No `python3` deny.** Loomyard must develop Python in other repos, whose tests run Python, so the agents' settings never deny it.
  What gets fixed is an agent reaching for Python where it did not need it.
- **Contract-file findings (hub's recommendation, not contested).** The verify gate's fixer stencil forbids writing `outcome.yaml` and `summary.md`,
  and accept-audit accepts a fork-contract-write finding once Master has rewritten those files in a later turn or they are absent.
- **Reset scope.** Whether the reset verb covers Webster only, or fabric's manual weft reset (`git -C <weft> reset --hard origin/<branch>`) as well, is left to Discussion: decide from what the refusals in each module need.
  The decision above on `git reset --hard` holds either way.

## Webster verify gate waits forever when Master ends its turn with a hung background shell

From note `webster-gate-hung-shell` (issues #361).

Imported from [issue #361](https://github.com/Knatte18/loomyard/issues/361).

Seen 2026-10-04 in `board-model`: after the verify gate re-prompted Master at 17:07:33, Master's fixer fork fixed the failure and Master's turn ended at 17:11, but the step logged nothing more for 75 minutes.
Master's Stop events (`.lyx/shuttle/<guid>/events.jsonl`) all carried `background_tasks` with a shell in state `running`: `python3 - 2>/dev/null; cat > /dev/null <<'EOF' … EOF; grep …`, where `python3 -` sat reading stdin forever.
The gate apparently does not count a Stop with a running background task as a turn end, so one hung shell blocks the run indefinitely; an operator prompt at 18:25 produced another such Stop and changed nothing.
Killing the `python3` process let the gate rerun verify at 18:26.

Expected: a turn end whose only running background task is a shell (not a fork) still counts, or the gate bounds the wait and surfaces "turn ended with background task X running" to the driver, which today cannot see or stop it (ly-drive has no verb for it, and Webster's interrupt policy is handback).
Also: lyx agents should not run `python3` at all (operator rule); the agent settings shuttle writes could deny it.

## accept-audit can never accept a fork-contract-write finding

From note `webster-accept-audit-contract` (issues #363, #365, #360).

Imported from [issue #363](https://github.com/Knatte18/loomyard/issues/363).

Seen 2026-10-04 in `board-model`: the run-exit audit flagged the verify-gate fixer fork for writing `_lyx/webster/outcome.yaml` (and `summary.md`), Master's contract files; Master had rewritten both afterwards.
`lyx webster accept-audit` refused even after HEAD was reset to the last batch head and both untracked files were deleted (their state at that head): "cannot be checked: …/_lyx/webster/outcome.yaml, …/summary.md; way forward: reset the branch to the run's start commit … and run lyx webster run --fresh".
`checkSuspectPaths` treats weft paths as unverifiable, so a `fork-contract-write` finding can never be accepted; the offered way forward throws away every batch (10 here).
The hub cleared `pendingAuditFindings` in `state.json` by hand to continue.

Expected: a contract-file finding is acceptable once the contract files are absent (Master rewrites them at its next terminal turn), or the gate fixer is kept from writing them in the first place.
Related: #361 (the same run's gate hang).


### From issue #365

Imported from [issue #365](https://github.com/Knatte18/loomyard/issues/365).

`lyx webster accept-audit` refused to accept pending audit findings:

```
webster: pending audit findings cannot be accepted: cannot be checked: _lyx/webster/outcome.yaml, _lyx/webster/summary.md; way forward: reset the branch to the run's start commit <sha> with git and run "lyx webster run --fresh"
```

Problems:

- The refusal says the two files "cannot be checked" but not why (missing, unreadable, changed since the audit, or not matching an expected state), so the caller cannot tell what to repair.
- The only way forward offered is resetting the branch to the run's start commit and rerunning with `--fresh`, which discards the run's work.
  There is no lighter path, such as regenerating the two files or re-running the audit.

Provenance: task `board-model`, note `webster-refusal-accept-audit.md`.


### From issue #360

Imported from [issue #360](https://github.com/Knatte18/loomyard/issues/360).

### escalation-to-human: the run halted with no OnStuck target and needs a human

**Slug:** board-model
**Parent:** main

**State:** blocked
**Current producer:** Webster

**Error:**
```
run-exit audit found 2 correctness finding(s): fork wrote "/home/knatte/Code/loomyard-LYXHUB/board-model/_lyx/webster/summary.md" — outcome.yaml and summary.md are Master's own contract files; a fork writing either forges the run's terminal judgment; fork wrote "/home/knatte/Code/loomyard-LYXHUB/board-model/_lyx/webster/outcome.yaml" — outcome.yaml and summary.md are Master's own contract files; a fork writing either forges the run's terminal judgment; suspect paths: /home/knatte/Code/loomyard-LYXHUB/board-model/_lyx/webster/summary.md, /home/knatte/Code/loomyard-LYXHUB/board-model/_lyx/webster/outcome.yaml; way forward: reset the branch to the run's start commit with git and run "lyx webster run --fresh" and re-step the Webster row (lyx webster run), since nothing the run recorded can check /home/knatte/Code/loomyard-LYXHUB/board-model/_lyx/webster/summary.md, /home/knatte/Code/loomyard-LYXHUB/board-model/_lyx/webster/outcome.yaml
```

**History:**
- Preflight / done / 2026-10-04T13:39:14Z
- Loom-Preflight / done / 2026-10-04T13:39:23Z
- Discussion-Write / done / 2026-10-04T13:44:33Z
- Discussion-Bouncer / stuck / 2026-10-04T13:45:35Z
- Discussion-Burler / stuck / 2026-10-04T13:47:21Z
- Discussion-Bouncer / stuck / 2026-10-04T13:47:52Z
- Discussion-Burler / stuck / 2026-10-04T13:49:53Z
- Discussion-Bouncer / awaiting / 2026-10-04T13:50:28Z
- Discussion-Bouncer / stuck / 2026-10-04T14:22:08Z
- Discussion-Burler / stuck / 2026-10-04T14:23:49Z
- Discussion-Bouncer / stuck / 2026-10-04T14:24:28Z
- Discussion-Burler / stuck / 2026-10-04T14:25:42Z
- Discussion-Bouncer / done / 2026-10-04T14:26:11Z
- Plan-Write / done / 2026-10-04T14:33:18Z
- Plan-Bouncer / stuck / 2026-10-04T14:33:52Z
- Plan-Burler / stuck / 2026-10-04T14:37:55Z
- Plan-Bouncer / stuck / 2026-10-04T14:38:40Z
- Plan-Burler / stuck / 2026-10-04T14:40:05Z
- Plan-Bouncer / stuck / 2026-10-04T14:41:02Z
- Plan-Burler / stuck / 2026-10-04T14:41:55Z
- Plan-Bouncer / done / 2026-10-04T14:42:27Z
- Batchifier / done / 2026-10-04T14:42:36Z
- Webster / stuck / 2026-10-04T16:26:41Z

## Webster run-exit audit: way-forward text ends mid-sentence and repeats paths

From note `webster-audit-way-forward-text` (issues #362).

Imported from [issue #362](https://github.com/Knatte18/loomyard/issues/362).

When the Webster run-exit audit halts a run as `blocked` on forged contract files, the way-forward text it prints is garbled.

The halt note reads:

> way forward: reset the branch to the run's start commit with git and run "lyx webster run --fresh" and re-step the Webster row (lyx webster run), since nothing the run recorded can check `_lyx/webster/summary.md`, `_lyx/webster/outcome.yaml`

The final clause, "since nothing the run recorded can check <paths>", names no consequence and reads as truncated.
The same two paths are also printed twice: once in each finding and again as "suspect paths".
The way forward also tells the operator to run `lyx webster run --fresh` and then `lyx webster run`, without saying whether the second command is a separate step or a restatement of the first.

Expected: a complete sentence stating why the forged files cannot be trusted, and an unambiguous ordered list of commands.

Provenance: task `board-model`, note `loom-halt.md`.

## No recovery path needs a command the agents are denied

From note `no-denied-recovery`.

Seen 2026-10-03 in the `pair-lifecycle` run: Webster refused a plan fix for a failed batch.
Its only named way forward was "reset the branch to the run's start commit with git and run `lyx webster run --fresh`" (see #350).
The driver, even in bypass mode, could not run `git reset --hard`, because `~/.claude/settings.json` denies `Bash(git reset --hard:*)` and a deny rule holds in every mode.
The run stopped until the operator ran the reset by hand.
`contracts/specs/refusal-spec.md` prescribes `git reset --hard <pre-fix head>` for another halt as well.

Idea (operator): a permission refusal must never stop the flow.
Every recovery a lyx refusal names must be doable by the agent that receives it.
Either lyx performs the destructive step itself, in Go, guarded to the run's own branch and start commit (`webster run --fresh` resetting the branch, a `keep|drop` verb for the integration fix commits), or the agents' permission set is loosened for what lyx's recoveries need.
Audit every refusal and stencil for commands the deny list blocks.
Related: `orch-live-hardening` (permission modes mean what they say).
