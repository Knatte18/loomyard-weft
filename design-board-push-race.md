# A rejected board push is retried after a fetch, and called a divergence only when one exists

`board-push-race` · Task · bug, fabric · running · Drift-Verify

## Seen (operator and hub orch, 2026-10-30)

At 15:22 a push of the board's `main` was rejected and creel mail #75 from `ly:FABRIC` told the orch that "origin/main holds commits this branch lacks", with the way forward `git reset --keep origin/main` or `git merge origin/main`.
Only this hub writes to the board's remote, so origin could hold nothing the local branch lacked.
Origin's reflog shows the rejected commit `3b6ed10c` arriving by a successful push at 15:23:07 and the next commit `54901f76` at 15:23:39, and a fetch right after the mail found the local branch only ahead of origin.
The rejection was thus between two pushes from this hub that overlapped, and no side had diverged.
The advice in the mail would have dropped a local commit (`reset --keep`) or merged a history that did not exist.

## Design (operator, 2026-10-30)

- A push rejected as not a fast-forward is retried after a fetch: when the fetch shows origin holding nothing the local branch lacks, the push goes through, and no mail is sent.
- A rejection is called a divergence, and its mail sent, only when the fetch shows origin holding commits the local branch lacks.
- Find out which two processes of the hub pushed the board at the same time, and whether the board's commit and push should hold one lock, so two pushes never overlap.
