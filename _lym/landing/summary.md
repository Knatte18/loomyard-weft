# Research: list the verbs and config keys no recent run used

Adds a candidate list, not a removal: the board entry asks the operator to choose which features go, so no code changes.

## What changed

- `docs/research/cleanup/unused-features-2026-11-18/report.md` lists the `lym` verbs with no use in Claude transcripts since 2026-10-28,
  split into verbs only their own package mentions and verbs a stencil, recipe or refusal names but no session ran.
  It adds the verbs with one to five uses, the evidence for the three features the `landingshed` reading named,
  and the config keys no layer of this hub overrides.
- `collect.sh` beside it reproduces the verb counts from the transcripts for any start date.
- `docs/research/README.md` gains a `cleanup` topic that links the report.

## Findings

- Of the verbs counted, 34 have no transcript use; 13 groups of them are named by nothing outside their own package.
- Of the three features the entry named, the flaky rerun fired in 8 of 151 landings since 2026-11-10 and the beam landing record is in use
  (`lym beam land` 24 uses, `lym beam describe` 24).
  The adopter-first repair note leaves no record to count, so the report gives no verdict on it.

## Design decisions

- No flag list: counting flags needs per-flag parsing of command lines, and the operator picks verbs first.
- The count cannot see an operator's own terminal, Codex sessions, or a verb a Go caller spawns; the report says so beside each table.
- The record held no decisions beyond the board body, which asks for a list, so nothing is removed.
