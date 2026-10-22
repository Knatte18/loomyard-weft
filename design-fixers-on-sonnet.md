# Fixers and other checked roles on Sonnet

`fixers-on-sonnet` · Note · enhancement, burler, claude

## Seen

`docs/research/token-audit-2026-10-22`: Opus and Sonnet read cache at the same price, so a swap saves about half a role's non-cache-read cost.
Roles whose output a later step checks: Plan and Discussion fixers about $162 over two weeks, darn writer $45, alignment $40, Webster fixer $37; together about 4% of spend.

## Idea (orch, not decided)

Run the fixers on Sonnet first, since the judge and the next round re-check them.
A3 (`review-setup-audit`) notes a sloppy fixer causes many late-round defects, and `docs/research/model-downgrade-results.md` holds earlier downgrade results; bench before switching.
