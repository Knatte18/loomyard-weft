# Orch instructions in two layers: lym's role and the repository's rules in heddle

`orch-instructions` · Project · enhancement, orch

## Goal (operator and orch, 2026-11-01)

Orch instructions live in two layers, each in one place:
- The orch role lym ships, the same for every orch in every repository.
- The rules for one repository's orchs, kept in heddle and managed with `lym orch rules`.

The orch note holds run state only, as its template's three sections say.

## Seen

The orch's instructions had spread over four places that disagree: the shipped role stencil, the hub-wide rules file on Bolt, an unused per-orch rules file, and standing rules the orch copied from note to note.
Rules meant for every orch sat in the loomyard hub's rules, and loomyard-only rules and code invariants sat only in the orch note.

## Bounds

The members change different packages and run as separate tasks; this project only groups them.
Once the heddle target and the role have landed, the orch moves the remaining loomyard-only orch rules into heddle through `lym orch rules`; that move is no task.
