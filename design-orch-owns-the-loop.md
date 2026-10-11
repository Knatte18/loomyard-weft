# Orch loop: one land command and orch start --continue

`orch-owns-the-loop` · Note · feature, batten, orch

## Goal

An orch session runs the whole loop with lyx commands and its committed role prompt.
Detached batten starts, run notices and the committed role prompt have landed; two pieces remain.

## One land command

The steps after a landing (pull in the prime, deploy, config reconcile, intake, prune, the batten teardown check, the fabric commit) become one lyx command.
Deploy stays operator-triggered, so the command either stops before deploying or deploys only when told.

## Continue the last orch session

After a reboot the operator wants the same conversation back, as `claude --continue` gives, not a fresh session reading a handoff.
Today that takes `lyx orch start --adopt <session-id>`, with the id dug out of `~/.claude/projects/`.
`lyx orch` records its session id and offers `lyx orch start --continue` (or makes it the default for a dead strand) to adopt the last orch session.

## Done when

- The orch lands a run with one lyx command.
- `lyx orch start --continue` after a reboot resumes the last orch conversation.
