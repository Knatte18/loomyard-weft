# Seed-IDE fails the run when the task branch's detached push has not landed yet

`seed-ide-push-not-landed` · Note · bug, batten, fabric

Imported from [issue #218](https://github.com/Knatte18/loomyard/issues/218).

Import note: the issue names files under `.lym/`, which is deleted with its worktree, so only what the issue quotes of them reached the board: `.lym/logs`.

## What happened

Run `weekly-time-report` (recipe raddle, promoted from a Sketch) stopped at Seed-IDE with:

> run "weekly-time-report" cannot act: the code tip e7b0cc2 the run's last transition names is not on origin yet; way forward: run "lym batten run weekly-time-report" again once the code push lands

Timeline from `.lym/logs` (2026-11-24, CET):

- 14:45:52 Worktree-Create done; the task branch forks from the prime's local `main` at e7b0cc2, one commit ahead of `origin/main`.
- 14:45:52 and 14:45:55 fabricengine spawned detached pushes of the new pair (`fabric --warp-path …/weekly-time-report …`).
- 14:45:57 Seed-IDE's bootstrap checked whether e7b0cc2 is on origin, found it was not, and refused the step.
- Shortly after, the detached push landed: `git ls-remote origin` showed `refs/heads/weekly-time-report` at e7b0cc2.

The network was slow and resetting that day (`Recv failure: Connection reset by peer` on records pushes), so the detached push took longer than the 2–5 s gap.

## Effect

- The run stopped as failed, and the tackler handed it back to the parent asking whether to push the prime's `main`, which was not needed.
- A plain `lym batten run weekly-time-report` afterwards went straight through Seed-IDE to Run-Shed.

## Expected

A check that the code tip is on origin, right after lym itself spawned the push that puts it there, should wait for that push, or retry for a while, instead of failing the run.
The stop should also say that lym's own push of the task branch is in flight, so the tackler does not ask the parent to push the parent branch.
