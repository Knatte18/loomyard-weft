# Convergence: log hub interventions and driver repairs per run by class

`convergence-plan` · Note · enhancement, orch, shed

## Seen

Operator 2026-09-30: take a step back and plan for convergence, not keep accepting more bugs.
That day nearly every hub intervention was in same-day new code (#319: approve nil dep, reject reading its file argument as run-id), and both passed tests built on fake deps.

## Idea

Log every hub intervention and every driver repair per run with a class and a cause, so interventions per run and new versus repeat failure classes are visible over time.
Fixing at the source with a test through the real CLI wiring, and hardening-only periods when the rate climbs, are working practice and need no build.

Goal: the outside LLM steps in rarely, and for the task rather than for lyx.
