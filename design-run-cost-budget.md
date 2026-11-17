# A per-run token and cost budget, audit mode first

`run-cost-budget` · Note · feature, shed, report · done

## Seen (orch, 2026-11-17)

Lym measures cost per phase (`f2295ed6`, `e62ce851`) and caps memory and time per run, but no run has a token or cost budget.
With Codex as a second engine and Weaver's model escalation, spend per run varies more than it did.
Source: the survey F17 (`orchestrator-survey-ideas`), from Fusion's per-task budgets and Conductor's `limits.budget_usd` with `budget_mode: audit|enforce`.

## Idea

- A per-run token or cost budget, set per profile or per task.
- It starts in audit mode: a run past its budget warns the orch once and goes on.
- An enforce mode, chosen later, pauses the run with a named reason.
