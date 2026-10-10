# Fixer report — plan, round 1

## Fixed

- **F1** (MEDIUM, design): the settle bullet had no stated bound for a failed or written-off advisor.
  Validated against `contracts/stencils/seat/seat-directive-chair.md:12` and `:20-21`, which name advisors that never started, the failure notice and the write-off, and tell the chair to decide without that advisor.
  In card 10's `**Intent:**`, the rewritten second bullet now says an advisor that never started, sent a failure notice or was written off counts as giving no answer.
  A new sentence states what the change lets through, a question settled without such an advisor's answer, and names the bound: the kept write-off bullet and the seat directive's decide-without-it rule.
  The `must` string `after weighing every advisor's answer` stays verbatim in the bullet, so the planned claim still matches.

## Disputed

None.

## Deferred

None.
