# Audit the use of the twill and lens review setup at every perch

`review-setup-audit` · Note · audit, burler, loom, report

## Seen (2026-10-18)

- On 2026-10-16/17 every perch got the reworked review setup: a lens fan in round 1 (`review-discussion`, `review-plan`, `review-code` beside a plain reviewer), a twill fixer every round, and twill writers for Discussion and Plan.
- The setup costs tokens and time at every perch, and no measurement yet shows what it catches that the plain setup missed.
- `plan-writer-finding-classes` classifies Plan findings to improve the Plan writer; it does not judge whether the setup is worth its cost.

## Operator decision (2026-10-18)

- Audit the use of the twill and lens setup across every perch once it has run for a while, and set the standard review rounds from the result (see `run-profiles`).

## Orch recommendation, not a decision

- Run it after about ten loom runs with the setup, from `lyx report convergence` (stop classes, fresh-code fixes, review rounds per segment), `tools/tokencount/tokencount.sh` and the review files in the records repository.
- Per perch: how many findings each fan member and round raised, how many led to a change that stayed, how many the later perches or PR-Gate would have caught anyway, and the tokens and minutes each round cost.
- Compare with the runs from before 2026-10-16 where the records allow it.
