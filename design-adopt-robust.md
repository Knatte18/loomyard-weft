# Adopt as one resumable algorithm that never starts work on its own

`adopt-robust` · Task · bug, enhancement, batten, tender, dobby, orch, fabric, shortcut

## Goal

Taking a hub or beam over from another machine is one predictable algorithm that never starts work on its own,
survives the session that started it, and resumes after a break.

## Findings (2026-11-02 and 2026-11-09)

- `lym adopt` run from the orch session died with it:
  the reconcile pulled the owner record naming the other machine,
  the orch watcher on this machine exited with `watcher_exit: owned by machine <other>`,
  and took the session and adopt with it, mid-deploy and before the owner write.
- A green adopt check lifted its holds and dobby started every adopted run,
  even after the orch set `lym batten pause` and `lym tender pause` on them; it happened on both dates.
- `lym tender pause` on an adopt-held run answered `already_paused: true` and recorded no pause of its own.
- An adopted run cannot be paused before adopt adds its pair: the pause verb refuses with `has no pair in this hub`.
- The local dobby started `batten run <slug> --continue-pass` on a run while adopt was taking it over.
- Adopt always starts the orch; the operator wanted to resume the orch session by hand.
- An interrupted adopt leaves no account of which steps ran; the operator had to run it again from a shell.
- The adopting machine cannot see the old owner's unpushed local commits.

## Proposed algorithm

Principles:

- Adopt never starts work: every adopted run stays paused until the operator resumes it, and only the operator lifts that pause.
- Adopt lives apart from the session that started it.
- Each step is recorded when done, and a second adopt resumes at the first step not done.

Steps:

1. Detach and record: adopt re-runs itself as a detached process and writes a local in-progress record naming its subjects.
   While it exists, the orch watcher does not exit on a foreign owner, and dobby starts no pass on a run under the subjects.
2. Reconcile from origin and check this machine's local commits origin lacks, as today, with keep-local and drop-local.
3. Add the pairs this machine lacks.
4. Pause every adopted run with an ordinary operator pause, never a hold that lifts by itself.
5. Write this machine as the owner, before the deploy, so the window in which the watcher sees a foreign owner is short.
6. Deploy when lym is behind.
7. Start dobby; start the orch only when asked: a `--no-orch` flag, and no orch start when adopt was run from an orch session.
8. Run the adopt check: green only reports and lifts no pause; red files a fix task as today.
9. Mark the adopt done; `lym beam status` shows an unfinished adopt and the step it stopped at.
10. The operator resumes the runs with `lym tender resume`.

Entry point:

- `lym s` (beam start, then reed attach) on a machine that does not own the worktree adopts it first:
  the beam in a beam, root in the prime, then starts and attaches.
  So adopt normally starts from a plain shell, never from the orch.

The old owner:

- When the old owner's dobby sees that another machine owns a beam or root,
  it pushes every local tip under that subject that origin lacks to an archive branch on origin, then stops its runs there.
  The new owner sees the archive branches and decides what to bring in.
