# lyx gate test --upto N with manual tiers, and a lyx gate status verb

`gate-tier-verbs` · Task · enhancement, gate

## Seen

Running a hub's tiers up to some level means naming each one: the loomyard push gate is `lyx gate test --tier 0 --tier 1 --tier 2`.
`--all` already means every package, not every tier.
The loomyard hub's `tier3` runs real LLM sessions, spends tokens and is run by hand only, so a plain "every tier" flag would make it easy to start by accident.
`lyx gate test` is the gate module's only verb: nothing shows which gate slots are taken, so a run waiting for a slot gives no reason.

## Operator decision (2026-10-22)

- `lyx gate test --upto N` runs every declared tier numbered 0 through N, in ascending order, dropping a tier another selected tier `covers`, as `--tier` does today.
- A tier may carry `manual: true` in `gate.yaml`; `--upto` skips a manual tier, which runs only when named with `--tier`.
- No separate "every tier" flag: `--upto` with the highest number, manual tiers left out, serves that need.
- `lyx gate status` is a read-only verb that shows which slots of the hub's gate-slot pool are taken, by which run and tier, and since when.

## Out of scope

Setting `manual: true` on the loomyard hub's `tier3` is a hub config write the orch makes after landing, not part of this task.
