# One run FSM over tracked records, and the orch resumes its own session

A run's lifecycle state is derived by one package, `internal/runfsm`, from tracked records alone,
with cancel and pause intents, the PR record and the fixer counts tracked.
An orch start ends the orch's own earlier Claude process and resumes its session on the same machine,
never launching fresh over a live holder.
`docs/research/loom/adopt-build-revision-2026-11-16/report.md` records how the rest of the adopt redesign reads against these two parts;
that rest is the project `adopt-build-resplit`, which builds on this branch after it lands.

## Members

- 3bc33584dbf8 The orch ends its own earlier process and resumes its session
- 973c9025036f One run FSM over tracked records, with cancel and pause intents
- #131 Report: revise the remaining adopt-build members against what landed
