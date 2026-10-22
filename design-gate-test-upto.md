# lyx gate test --upto N, and manual tiers it skips

`gate-test-upto` · Note · enhancement, gate

## Seen

Running a hub's tiers up to some level means naming each one: the loomyard push gate is `lyx gate test --tier 0 --tier 1 --tier 2`.
`--all` already means every package, not every tier.
The loomyard hub's `tier3` runs real LLM sessions, spends tokens and is run by hand only, so a plain "every tier" flag would make it easy to start by accident.

## Idea

- `lyx gate test --upto N` runs every declared tier numbered 0 through N, in ascending order, dropping a tier another selected tier `covers`, as `--tier` does today.
- A tier may carry `manual: true` in `gate.yaml`; `--upto` skips a manual tier, which runs only when named with `--tier`.
- No separate "every tier" flag: `--upto` with the highest number, manual tiers left out, serves that need.

## Operator view, not a decision

The operator asked for an "every tier" flag and a "tier N and every tier below" command; the orch recommended `--upto N` plus `manual` instead of an "every tier" flag, and the operator agreed to record this note (2026-10-22).
