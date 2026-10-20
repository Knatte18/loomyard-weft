# Agents commit code through lyx fabric commit, not raw git commit

`agents-commit-through-fabric` · Note · enhancement, fabric, stencil

## Seen (orch, 2026-10-20)

The darn writer (`contracts/stencils/darn/darn-template-write.md`) and webster's implementer and verify-fix forks (`webster-body-implementer.md`, `webster-body-verify-fix.md`) commit their code on the task branch with plain `git commit`; Go commits only the `_lyx` run state.

## Operator view (2026-10-20)

`lyx fabric commit` was made for exactly this purpose, so agents should commit through it rather than through raw git. Not urgent.

## Idea

The committing stencils name `lyx fabric commit`, which then needs to commit warp code changes as well as weft, keep refusing `_lyx` paths in a code commit, and take the subject the stencil prescribes (`N: <name>`, `fix: <summary>`).