# A dedicated agent spawned by the run or dobby settles a review escalation, not an orch fork

`escalation-decider-agent` · Note · feature, burler, arbiter, dobby, orch

## Seen (operator and orch, 2026-11-01)

A review escalation (a Discussion or Plan bouncer at its hard cap, or a fixer's decision findings) mails the orch, whose procedure reads the brief through a one-shot fork of its own session and then records the answer with `lym loom decision add`, `lym loom circling accept|continue` and `lym loom resume`.
On 2026-11-01 this ran four times in a few hours for `plan-split-producers` and `reed-status-buttons`; each fork inherited the orch's whole context, and every call sat with the orch while it also talked with the operator.
The calls themselves were run-level: whether a fix in the record holds, whether to accept at the hard cap, small scope rulings inside the board design.

## Idea (operator, 2026-11-01)

A dedicated agent settles the escalation, spawned by the run itself or by dobby, never a fork of the orch.
- It starts fresh with the brief, the decision record, the review's findings and the board entry's design, and reads the code it needs.
- It decides the run-level calls, records each with `lym loom decision add`, chooses accept or continue and resumes the run.
- It passes to the operator only a question the board entry's design does not settle, through the orch or straight to the operator, as the run's other operator questions go.
- The orch hears only the outcome, so a quiet orch, as in note `dormant-orch`, need not wake for it.

## Open

- Whether the run spawns it at the bouncer's escalation row, or dobby does on the escalation notice.
- How it relates to the arbiter, which already settles some halted stops for the parent.
- What it may decide alone: within the board entry's design and the record's decisions, or wider.
