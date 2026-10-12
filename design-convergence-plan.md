# Convergence report: count orch wakes, driver repairs and fresh-code fixes from existing records

`convergence-plan` · Task · enhancement, orch, shed · running · PR-Gate

## Seen

Operator 2026-09-30: take a step back and plan for convergence, not keep accepting more bugs.
That day nearly every hub intervention was in same-day new code (#319: approve nil dep, reject reading its file argument as run-id), and both passed tests built on fake deps.

An audit on 2026-10-12 found no metric to judge convergence by: the week of 2026-10-05 had about one landing in three a lyx bug fix, roughly half of those repairing code landed within three days, while new features more than doubled.
The goal for the orch (operator, 2026-10-12) is to be disturbed less: most of a run should stay inside loom and its worktree, which `alignment-agent` and `orch-split` work toward, and nothing measures whether they do.

## Idea (revised 2026-10-12)

Count from records that already exist, with no new bookkeeping by the orch or a driver: a hand-kept class per intervention costs agent turns and is kept unevenly.

A `lyx` report verb counts per run and per week:

- orch wakes: batten notices sent to the orch, parent-review requests and escalations, by kind;
- driver repairs and stops: from the drive reports, by stop class, and arbiter calls;
- hand steps: goto, resume and hand landings recorded in the run's history;
- parent decisions added after Discussion;
- review rounds per segment (Discussion, Plan, Webster and merge review) and their outcome, so a change meant to save rounds shows (from `twill-plan-followups`: does twill for Plan-Write cut Plan-Review rounds; also `webster-pack-read-set`).

From git on the parent branch it counts landings by kind and the share of fixes that repair code landed within a few days (the fresh-code share).

Questions the report answers:

1. Do orch wakes per run fall as `alignment-agent` and `orch-split` land?
2. Do the same stop classes and failure classes come back run after run?
3. Does the fresh-code fix share fall?
4. Do review rounds per segment fall after a change meant to cut them?

Fixing at the source with a test through the real CLI wiring, and hardening-only periods when the rate climbs, are working practice and need no build.
Goal: the outside LLM steps in rarely, and for the task rather than for lyx.
