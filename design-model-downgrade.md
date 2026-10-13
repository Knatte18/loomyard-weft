# Model downgrade per role

`model-downgrade` · Note · enhancement, burler, loom, webster, shuttle

## Idea (operator, 2026-10-13)

Lyx spawns many agents a day, and most roles run on Opus or Sonnet.
Find the roles that can move to a cheaper model (Sonnet 5.5 or Haiku 5.5) without losing quality.

## First step: a bench (operator, 2026-10-13)

Take a few older runs that have landed and replay the same run with the cheaper model in a role, everything else the same.
Compare cost, turns, rounds, findings, stuck batches and the outcome against the original run.
LLM runs vary, so run each arm more than once before reading a difference into it; the `twill-plan` bench (`.scratch/twill-plan-bench/` in the hub prime) is the precedent for the setup.
No model setting changes before the bench has shown it holds.

## Study (2026-10-13)

Report `.scratch/model-downgrade.md` in the hub prime: every role with its model, where it is set, its cost share over the six runs in `.scratch/token-usage-by-role.md`, a proposal and the risk.
Savings below are estimates from those six runs, not measured.

- Cost is dominated by cache reads in long sessions, and Sonnet 5.5 reads cache at the same price as Opus ($0.20 per million), so Opus to Sonnet saves about 30-40 % of a role's cost; the large saving is Haiku.
- Log reading is already cheap: the hub's driver runs on Haiku and cost about $0.20 over the six runs.
- Candidates, ranked by estimated saving, all config-only:
  1. Plan-Review and Discussion-Review reviewers to `sonnet[high]` from round 2, keeping Opus for round 1: about $10.
  2. Bouncer judge and seed to `haiku[medium]`: about $7.5; check the judge for false CONVERGED verdicts.
  3. Plan and Discussion fixers to `sonnet[medium]`: about $4.
  4. Small items under $1 each: conflict resolver, friction, describe.
- Keep Opus for plan, discussion, arbiter, alignment and recovery.

## Webster

The study's largest lever was the Webster Master on `haiku[high]` (about $30), but its implementer forks inherit the Master's model and write the code.
The operator found Haiku (an older version, in Millhouse) often too weak as an implementer (2026-10-13).
Haiku 5.5 is cheap only below 100k context, and Webster forks run at 50-410k, so they often pass it.
A per-batch model (a cheap model only for mechanical cards) needs code in `internal/websterengine`.

## Per-round models and the fan (operator, 2026-10-13)

The operator wants each review round's reviewer and fixer pair able to run on its own model, for instance a cheap model first to fix the coarsest findings, since the Burler fan is built to catch broadly even on weaker models.
Per-round models exist today: `review`, `fix` and each segment's `*_review`, `*_fix` and `fan_review` take a YAML list with one model per round, the last entry serving every later round.
`plan_fan` and `discussion_fan` turn the fan on per segment; empty runs solo.
Fan on or off per round (for instance fan in round 1, solo after) does not exist and needs code.
Open question for the bench: whether the fan pays at all, against a solo reviewer; arms such as solo Opus, fan Sonnet and fan Haiku on the same plan.
Risk to measure: round 1 finds the most, so a weak first round pushes findings to later rounds with less budget left.

## Code items

- Fork the fixer from the reviewer's session so it does not read the context again (part of the $1.9 per run in `.scratch/burler-cost-breakdown.md`); skipping the fixer on an empty review is already built (`internal/burlerengine/fix.go`).
- A separate model for twill lens forks, parent-review forks and Bouncer escalation forks is not possible as forks: they inherit the parent's model, and a non-fork subagent loses the shared cache prefix.
