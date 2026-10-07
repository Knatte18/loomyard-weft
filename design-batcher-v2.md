# Batcher v2: Merriam context at fork time, a fitted model, plans in card order

`batcher-v2` · Task · bug, enhancement, webster · running · Plan-Write

Card batching (landed 2026-10-07) cut Webster's time and tokens visibly; this task makes its model measured instead of guessed.

## Merriam's context at fork time

Operator, 2026-10-07: the cost batchifier's `startup_context` (60000 in the `cautious` profile) is a fitted constant standing in for something that is not constant.

An implementer fork is a fork of Merriam, so it starts with Merriam's whole context at the moment it is spawned, and that context grows with every finished batch.
A constant is too high for early batches and too low for late batches of a long plan, which is where a large batch can overrun the budget.

The start context of batch N is Merriam's base context plus a growth term for batches 1 to N−1.

- Base: computable exactly before the run, since it is fixed by what Merriam loads: Claude Code's system prompt and tools, CLAUDE.md, the Webster stencil with the inlined PATTERN overview, and the plan files Merriam reads.
- Growth: what comes back between forks, the batch reports and the output of Merriam's `lyx webster` verbs; estimated, for example as a per-card or per-report allowance.

The partition stays computed once per run, as `webster-spec.md` requires.

Measure first: read Merriam's actual context at each fork from its transcript in the runs that used the `cautious` profile (`shuttle-turns`, `test-tiers`, `run-recovery`), to size the base and the growth term and to see how far the constant was off.
Compare weight per card in those runs against earlier identity-batchifier runs with `tools/tokencount`.

## Learning loop

Operator, 2026-10-07: the batchifier's model is a first guess; treat tuning as a learning loop rather than a design-time decision.

- Log: each batch records its profile, coefficients and per-card estimate components in webster's `state.json`; the fork transcripts give the actual peak context (max input+cache tokens of any message) and the actual weight.
- Analyse: extend `tools/tokencount -calibrate` to read those records from finished runs and fit the peak-context coefficients (estimate vs actual peak) and the cost side (weight per batch vs batch size).
- Decide on data: the batch size and budget that minimise weight per card without a rise in failed batches, recoveries or Webster-Review findings; quality outcomes (batch failed, recover-batch, review findings per card) belong in the same table.
- Re-fit as runs accumulate, and compare profiles across runs before changing the default.

## One flag for the weft repository

`tools/tokencount` has both `-weft` (finished runs) and `-history` (calibration) naming the same weft repository; one flag with one default would do.

## Plans run in card order

Rule (operator, 2026-10-06): a valid plan is executable strictly in card-number order; a card never depends on anything a later card does, and Webster follows that order as written and never sorts cards.

Plan-Gate refuses backward dependencies since `bugfix-webster` (`uses-later-target`, `delete-before-reference`), and card-batching made the batchifier's contiguous segments in card order the execution order.
Still open: `contracts/specs/loom-plan-spec.md` calls the card list a DAG of intent "not itself an execution order", and the plan template does not tell the writer the rule.
Both state it, to the plan writer and the plan reviewer, so Plan-Write stops bouncing on plans Webster used to reorder.

## Card commands that reach the fabric repo

Operator, 2026-10-07: folded in while the run was in Discussion.

Seen 2026-10-07 in card-batching: card 08 (`_lyx/plan/08-calibrate-template.md:16`) ordered `go run ./tools/tokencount ... -history ../card-batching-weft`. The fork obeyed, record-batch failed the audit with `fabric-reference`, `recover-batch` refused `needs_fresh` and `accept-audit` refused too, leaving only a full `reset --to start`.

The plan validator should refuse a card command that names a path outside the tracked tree (the weft repo above all), so Plan-Review never passes one. Separately, a `fabric-reference` from a batch with no commit and HEAD unchanged cannot have touched the tree; `accept-audit` could accept it.
The no-commit case has since landed (`accept-audit --batch`, 83a7d6691).

Seen again 2026-10-07 in shuttle-turns, batch 13 (cards 13–16): a fork, most likely for `15-replay-corpus`, read other worktrees' shuttle events (`*/.lyx/shuttle/*/events.jsonl`, `*-weft/.lyx/shuttle/...`) with `cat`/`jq` to build a corpus, then committed all four cards.
The findings were read-only and pathless, but the batch had commits, so `accept-audit --batch` refused and the hub had to `git reset --keep` the batch's start and redo all four cards.
Two wants:

- `accept-audit --batch` accepts pathless `fabric-reference` findings on a batch with commits when the findings come from read-only commands, or the audit tells a read from a write in the first place.
- The plan stencil steers a corpus or fixture card to committed test data, never to live files of other worktrees.


## From issue #424

Imported from [issue #424](https://github.com/Knatte18/loomyard/issues/424).

## What happened

In one loom run, webster batch 08 failed `lyx webster record-batch 8` three separate times with the same `batch_failed` cause.
Each time the implementer fork ran `go run ./tools/tokencount -last 40 -calibrate cautious -history <sibling fabric repo path>`, and the audit reported:

`fabric-reference violation ... ran a fabric-referencing command (...) that can rewrite run state — an implementer fork must never touch the fabric repo directly`

Each failure was followed by `lyx webster recover-batch 8`, which refused with `needs_fresh: true` ("batch 08 failed on findings recovery cannot check: fabric-reference ... (the finding names no path)").
The webster master then escalated to the human each time, so the loom run halted as `blocked` (anomaly `escalation-to-human`) three times.
A driver re-step after a `lyx` binary change (`last_step.binary_changed`) respawned the master, which asked again for the same cause, so the new build did not change the outcome.

## Why it is abnormal

- The fork repeats the same command in every attempt, so whatever instructs it (batch brief or plan) and the audit disagree; no attempt can pass, and the loop is only broken by a human.
- The audit treats any command mentioning the fabric path as one that "can rewrite run state", without distinguishing a read-only calibration invocation, and the finding carries no path, so `recover-batch` cannot verify it and refuses.
- The `recover-batch` refusal's way forward is inconsistent between attempts: the first offered only `reset --to start` then `run --fresh`, the later ones also offered `accept-audit --batch 8` then `recover-batch 8` (when the batch made no commit and the worktree is clean).
  A batch that failed only on an audit finding should say up front which of these applies.

## What to do

- Decide whether `tools/tokencount` with `-history` is a legitimate implementer-fork command; if so, whitelist read-only tokencount calibration in the fabric-reference audit, or make the batch brief tell the fork not to run it.
- Make the audit explain what the fork should do instead, and have `recover-batch` offer `accept-audit` consistently whenever it is applicable.
- Consider a halt that repeats the identical audit finding to be surfaced as one escalation with the cause, rather than three master questions.

Provenance: task `card-batching`; notes `webster-refusal-record-batch.md`, `webster-refusal-record-batch-2.md`, `webster-refusal-record-batch-3.md`, `webster-refusal-recover-batch.md`, `webster-refusal-recover-batch-2.md`, `webster-refusal-recover-batch-3.md`, `loom-halt.md`, `loom-halt-2.md`, `loom-halt-3.md`, `driver-binary-change-restep-20261007.md`.


## From issue #432

Imported from [issue #432](https://github.com/Knatte18/loomyard/issues/432).

## What happened

A webster batch (`13-tail-in-stencils-and-steer`) needed run-event data that lives under the weft checkout's `.lyx/shuttle/*/events.jsonl`.
The implementer fork read those files with read-only shell commands (`head`, `jq`, `awk`, `cat`, `grep`).
`lyx webster record-batch 13` failed the batch with a fabric-reference violation for each such command ("can rewrite run state — an implementer fork must never touch the fabric repo directly").
The batch's card did not say which route to the data is allowed, so the fork had no way to know the reads would be flagged.

The way forward then looped:

1. `lyx webster recover-batch 13` refused: "batch 13 failed on findings recovery cannot check", with `needs_fresh: true` and the finding "names no path".
   It was run twice with the identical refusal.
2. The refusal pointed to `lyx webster accept-audit --batch 13` followed by `recover-batch`, valid only when the batch made no commit.
3. `lyx webster accept-audit --batch=13` refused: "batch 13 made a commit or recorded no start commit, so its tree cannot be shown unchanged"; its way forward is `lyx webster reset --to start` and `lyx webster run --fresh`.

The batch had committed work, so the only remaining route was discarding the batch.
A recovery strand later read the same files read-only through the worktrees' own `.lyx` links (which name no weft path) and the audit passed, so the route was found by trial.

## Why it is abnormal

- The audit classifies commands that only read run files as ones that "can rewrite run state"; a read-only `head`/`jq`/`cat` of an events file is not a rewrite.
- The finding carries no path, so `recover-batch` cannot verify it and demands a fresh run, and `accept-audit` is unusable once the batch has committed.
  A finding that is a false positive on a committed batch has no accept route.
- Nothing tells an implementer (card, prompt or audit message) which access route to run files is permitted.

## What to do

- Make the fabric-reference audit distinguish read-only commands from ones that write, or name the allowed read route in the implementer prompt/card.
- Let `accept-audit` (or `recover-batch`) handle a finding on a batch that committed, for example by checking that the commit does not touch the fabric repo, instead of requiring an unchanged tree.
- Give the finding a path (or the command's target) so recovery can check it.

Provenance: task `shuttle-turns`; notes `webster-refusal-record-batch.md`, `webster-refusal-recover-batch.md`, `webster-refusal-recover-batch-2.md`, `webster-refusal-accept-audit.md`, `13-tail-in-stencils-and-steer-recovery.md`.
