# Report the check of five redundancy candidates of internal/boardengine

Adds `docs/research/speed/redundant-candidates-check-2026-11-17/report.md`, linked from `docs/research/README.md`.
It records a run of `lym gate test --wait --map` and `lym gate redundant` on `internal/boardengine` after the kill-count fix,
and five candidates read by hand against their covering tests, before `redundancy-sweep-existing` relies on the list.
No code changes, and no test is deleted.

## What the trial found

- The map build took 10.2 s at 313 MB; `lym gate redundant` took 22 min at 2.1 GB.
- It listed 6 candidates, 8 unconfirmed, 29 refuted, 8 kept and 1 not judged.
  Every candidate and unconfirmed test is among the earlier trial's 14 candidates.
- Of five candidates read, 0 are true redundancies: each asserts something its covering tests do not.
  The assertions are the resolved template defaults, a beam's kind-only and beam-only change, the hold-free producers, the Sketch exemption's refused changes, and the raddle store's decoded fields.
- The mutation check passes them because no mutant changes a literal or deletes one clause of a boolean chain.

## Design decisions

- The five were picked from the 6 candidates, leaving `TestLoadConfig_DefaultRecipe` unread, since the list is too short to pick across.
- A verdict rests on the assertions, not on the verb's kill counts.
- Two notes filed instead of fixes: `redundant-mutants-skip-literals` and `redundant-clause-inputs-unfed`.

Created using
 - LoomYard https://github.com/Knatte18/loomyard
