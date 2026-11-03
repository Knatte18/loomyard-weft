# A run lands on a parent the hub may not push to by merging on the remote

`protected-parent-landing` · Note · feature, landing, git

## Seen (operator, 2026-11-03)

On the NORCE Models repository, on machine OSL-1046, a direct push to `main` is refused by branch protection: every merge into `main` has to happen on the remote.
That is intended and stays so.
lym lands every run by merging locally and pushing the parent: a task's Finalize squashes onto the parent and pushes it, and a beam's pawl run lands the beam the same way.
So on such a repository no task and no beam can land.

## Idea

A hub, or a parent branch, can be marked as landed through the remote:

- Finalize pushes the task or beam branch, opens a pull request into the parent and merges it on the remote, then pulls the parent, in place of the local squash and push.
- The parent lock, the gate of the merged tree and the board's done mark stay as today.
- A refused push to a protected branch names this setting as the way forward, instead of a bare push failure.

## Open

- What the protection demands beside no direct push: required approvals, required status checks or a merge queue, which decide whether lym can merge at once or must wait.
- Squash, merge commit or rebase on the remote, as the repository allows.
