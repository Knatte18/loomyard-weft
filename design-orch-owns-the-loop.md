# Orch owns the run loop: detached batten start, a lyx watch, one land command, --continue, a committed role prompt

`orch-owns-the-loop` · Note · enhancement, batten, orch

## Goal

An orch session runs the whole loop (start, watch, review, land, deploy) with lyx commands and its committed role prompt.
It needs no scripts in `.scratch/hub/` and no rules it must remember to read.

## Start a run detached

`lyx batten run <slug>` blocks in the foreground, so the orch backgrounds it from its own shell, and a Claude Code background job is killed at its time limit along with its process group: on 2026-10-02 all three batten runs died that way after 30 minutes.
The hub now starts them with `setsid nohup` by hand.
`lyx batten start <slug>` detaches batten's Go driver itself (`proc.Detach`, as `lyx loom start` does for loom's go driver).
How the operator sees many batten runs at once (a status strand, its height, or reed's own-window anchoring) is undecided and stays out of this task.

## A watch command

Done: `orch-cycle-hardening` landed batten run notices (`[batten notice]`) that wake the orch when a run stops, its driver dies or its status goes quiet; see `internal/battenshed/notice.go` and the package's `doc.go`.

## One land command

`.scratch/hub/land.sh <slug>` plus the steps after it (pull in the prime, delete the branch on origin, `lyx board prune`) become one lyx command.
Deploy stays operator-triggered, so the command either stops before deploying or deploys only when told.

## Continue the last orch session

After a reboot the operator wants the same conversation back, as `claude --continue` gives, not a fresh session reading a handoff.
Today that takes `lyx orch start --adopt <session-id>`, with the id dug out of `~/.claude/projects/`.
`lyx orch` records its session id and offers `lyx orch start --continue` (or makes it the default for a dead strand) to adopt the last orch session.

## The orch role prompt

Moved into task `lyx-skills-and-stencils` (operator, 2026-10-05).

## Done when

- The orch starts, watches and lands runs with lyx commands only, and `.scratch/hub/` is gone.
- A started batten run survives the orch's shell and session ending.
- `lyx orch start --continue` after a reboot resumes the last orch conversation.



