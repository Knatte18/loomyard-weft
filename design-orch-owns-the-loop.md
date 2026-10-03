# Orch owns the run loop: detached batten start, a lyx watch, one land command, --continue, a committed role prompt

`orch-owns-the-loop` · Notes · feature

## Goal

An orch session runs the whole loop (start, watch, review, land, deploy) with lyx commands and its committed role prompt.
It needs no scripts in `.scratch/hub/` and no rules it must remember to read.

## Start a run detached

`lyx batten run <slug>` blocks in the foreground, so the orch backgrounds it from its own shell, and a Claude Code background job is killed at its time limit along with its process group: on 2026-10-02 all three batten runs died that way after 30 minutes.
The hub now starts them with `setsid nohup` by hand.
`lyx batten start <slug>` detaches batten's Go driver itself (`proc.Detach`, as `lyx loom start` does for loom's go driver).
How the operator sees many batten runs at once (a status strand, its height, or reed's own-window anchoring) is undecided and stays out of this task.

## A watch command

`.scratch/hub/watch.sh <slug>` (one watcher per run) becomes a lyx command, or a part of orch's own watcher.
It wakes the orch session when a run changes to a state that needs it, as a backup to the driver's own message to its parent, which is the primary path.

## One land command

`.scratch/hub/land.sh <slug>` plus the steps after it (pull in the prime, delete the branch on origin, `lyx board prune`) become one lyx command.
Deploy stays operator-triggered, so the command either stops before deploying or deploys only when told.

## Continue the last orch session

After a reboot the operator wants the same conversation back, as `claude --continue` gives, not a fresh session reading a handoff.
Today that takes `lyx orch start --adopt <session-id>`, with the id dug out of `~/.claude/projects/`.
`lyx orch` records its session id and offers `lyx orch start --continue` (or makes it the default for a dead strand) to adopt the last orch session.

## The orch role prompt

The orch's working rules live in one committed role stencil beside the other orch stencils (`contracts/stencils/orch/`), so an orch on any machine works by the same rules; today they sit in one machine's file-memory and in handoffs.
`lyx orch` puts the role into its start prompt and points the one-line resume and adopt prompts at it.
The rules, one section per theme:

- Parent-review: a one-shot fork reads the brief, checks scope against the board entry only, and submits `lyx loom review approve|reject <slug>`; a repeat notice starts no second fork.
- PR-Gate: judge each run against its own board goal and its tests, never widening scope through rejects; fix a small goal defect on the task branch, and put new hardening on the board instead of rejecting.
- Board: read and write it only through `lyx board`; a proposal not yet shaped into a task goes on the board as a tier 3 note at once, never in a local draft file; fewer, larger tasks, since each run costs a full Discussion/Plan/Webster/review cycle; a small finding never becomes its own entry, but folds into an open entry whose work overlaps (same files, same context), or into the repo-wide conventions task when it overlaps nothing.
- Bugs in lyx go to GitHub issues; design, features and hardening go on the board.
- Act and fix without asking; ask the operator only about what to build.
- Give `cd <pair> && lyx reed attach` for every run in every status report.

The matching file-memory entries on the hub machine are deleted when the role lands.

## Done when

- The orch starts, watches and lands runs with lyx commands only, and `.scratch/hub/` is gone.
- A started batten run survives the orch's shell and session ending.
- `lyx orch start --continue` after a reboot resumes the last orch conversation.
- An orch cycle's resumed session knows the procedure from its role prompt, and its handoff carries state only.
- A fresh `lyx orch start` on a machine with no file-memory works by the same rules.

