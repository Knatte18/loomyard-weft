# Weaver: a cold-start implementer beside Webster, with model escalation

`weaver` · Note · feature, webster

## Idea (operator, 2026-10-11)

Webster stays fork-based: its implementer and verify-fix forks start at once with Master's context and run on Master's model.
Weaver is a second implementer beside it, built on Millhouse's "Builder" (skill `mill-go`), that runs a cold-start agent per batch, so each agent can take its own model, and escalates the model when an agent fails.

## Seen (research, 2026-10-11)

- Webster spawns one cold agent today: the recovery strand.
  When a fork reports `stuck` or writes no report twice, Master runs `lyx webster recover-batch` (`contracts/stencils/webster/webster-template-master.md`), which starts a fresh shuttle/reed strand with a full cold-start prompt (`internal/websterengine/recoverbatch.go`).
  Its model is `webster.yaml` `recovery` (default `opus[high]`), against `master` at `sonnet[medium]`; that is Webster's only model escalation, and a batch gets at most two recoveries.
- Builder (`plugins/mill/skills/mill-go-base/SKILL.md` in the Millhouse worktree) is a lean orchestrator that never reads cards or diffs.
  Per batch it starts a fresh, non-fork agent with a model from config (`roles.implementer.model`), and the agent reads only its batch file and the overview's Shared Decisions.
  It has no automatic model escalation: a stuck batch gets one fresh retry on the same tier, then a plan edit with a `## Prior failure` note, then a question to the parent.
- Millhouse's cold starts were slow: each agent read much background and loaded its tools before it began work.
  Likely further costs visible in the skill: a review loop per batch, finalize replaying every batch's verify in turn, and a state commit on every status change.

## Shape

Weaver is, in effect, Webster's recovery strand used as the normal path.

- From Webster: the Go bracket verbs (begin-batch, record-batch, recover-batch), the report contract and classification, suspect paths, partition and sequencing, the plan-level verify gate and failure ladder, the shuttle/reed spawn path, and `RenderRecoveryPrompt` with `uncommitted_paths` and the failure digest.
- From Builder: a cold, scoped agent per batch that reads only its cards and Shared Decisions, a model and effort per role per spawn, and a one-shot self-resolve before halting.
- New in neither: an escalation ladder per failed attempt, for example `sonnet` → `opus` → `opus[high]`, set in config.
- Open: how to keep cold-start cost down (a kick-start pack per batch, see `kick-start-pack`), and how a run chooses Weaver over Webster (recipe row or config).
