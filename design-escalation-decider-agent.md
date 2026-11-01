# The arbiter settles review escalations and records their decisions, so they never reach the orch

`escalation-decider-agent` · Note · feature, burler, arbiter, dobby, orch

## Seen (operator and orch, 2026-11-01)

Review escalations reached the orch four times in a few hours for `plan-split-producers` and `reed-status-buttons`, and the orch read each through a fork of its own session and recorded the answer with `lym loom decision add`, `lym loom circling accept|continue` and `lym loom resume`.
The calls were run-level: whether a fix in the record holds, whether to accept at the hard cap, small rulings inside the board entry's design.
The arbiter, a cold stronger-model agent the Go driver spawns at a stop it cannot settle (`internal/shedarbiter`), exists for such calls, but two limits sent them on:
- `internal/arbiterstop` makes only a review escalation of cause `hard-cap` or `circling` eligible; a `decision` escalation, such as a fixer's decision findings or a bouncer that wants the parent's ruling, goes to the run's parent without the arbiter.
- An eligible arbiter may only resume (a goto) or hand on: it records no decision and edits nothing.
  At `reed-status-buttons`' hard cap it found the false clause, wrote the exact three-line correction, and still had to hand on, since "I may edit nothing beyond this verdict, and no recorded operator decision covers it".

## Idea (operator, 2026-11-01)

The arbiter settles these calls itself, so they never reach the orch.
- A `decision` escalation becomes eligible for the arbiter as `hard-cap` and `circling` are.
- The arbiter may record a decision with `lym loom decision add`, under its own `--by`, and choose `circling accept` or `continue`, as the orch does today.
- It passes on only a question the board entry's design and the record's decisions do not settle; that goes to the operator through the orch.
- The orch hears only the outcome, so a quiet orch, as in note `dormant-orch`, need not wake for it.

## Open

- How far the arbiter may decide alone: within the board entry's design and the record's decisions, and where that line is drawn in its stencil.
- Whether `--by arbiter` is a new decision author, shown as such in the record.
