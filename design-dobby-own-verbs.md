# Dobby gets its own lyx dobby verbs, and lyx orch start stops starting it

`dobby-own-verbs` · Note · enhancement, orch, workspaces

## Seen (operator, 2026-10-25)

`lyx orch start` starts every dobby this machine owns before the orch session, and dobby's own verbs sit under `lyx beam` (`lyx beam start`, `lyx beam status`), whose names say beam where they mean dobby.
A hub without a running orch gets no dobby unless someone runs `lyx beam start`, and restarting the orch is mixed up with bringing the hub up.

## Idea

The two are tied but separate, and each gets its own verbs:

| | dobby | orch |
|---|---|---|
| What it is | a program, no LLM, one per worktree (the prime and each beam) | an LLM session, one per worktree thread |
| What it does | starts released ready tasks, drives their batten passes, lands them and cleans up | answers escalations and notices, carries the operator's answers, keeps the board |
| Without the other | runs start and land; a stop that needs a decision waits | nothing starts; the orch can still talk and write the board |
| Talks to the other | sends notices to the orch | never drives dobby |

- `lyx dobby start|status|stop` replace `lyx beam start` and `lyx beam status`; `lyx beam` keeps only what concerns beams: `create`, `promote`, `land`.
- `lyx orch start` starts only the orch session, and warns when the worktree's dobby is not running, naming `lyx dobby start`.
- `lyx prime`, the verb that brings the hub up, runs `lyx dobby start` and then `lyx orch start`, then attaches.

## Open

- Whether `lyx dobby start` starts every dobby this machine owns, as `lyx beam start` does, or only the current worktree's by default.
