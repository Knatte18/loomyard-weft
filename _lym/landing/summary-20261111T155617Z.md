# Adopt takes over through a pause, a short push lock and stale stops

This change fixes three ways a `lym adopt` takeover failed.

## A pause no longer holds an adopt

A hub-wide pause left from before a machine move made `lym adopt root` hang.
Its worker's first network call waited at the pause gate and wrote the line only to `adopt.out`.
An adopt is the operator's own takeover, so the caller (`runAdopt`) and the worker (`runWorker`) now call `pauserecord.SetGateExemptAll` on their own process's gate first.
This is the exemption `lym fabric remove` and `lym batten cancel` already use.
They reach it through a new `ExemptPause` seam on `callerDeps` and `workerDeps`.
The exemption covers only the adopt's own process.
A dobby or an adopt check that the worker starts installs its own gate, and a pause still holds it.

## The adopt's reconcile waits on a short push lock

Once the hub pause was lifted, the deferred records push it spawned held the weft push lock.
The adopt's reconcile read `main-weft (busy)` and stopped.
`ReconcileOptions` gains `PushLockWait`.
While a records branch's write lock is free and its push lock is held, `waitRecordsLocks` tries both again once a second until that bound has passed.
It holds no lock between tries.
The adopt passes `reconcilePushLockWait`, 30 seconds.
Every other reconcile passes zero and reads a held push lock as busy at once, as before.
A push parked under a pause still outlasts the bound and reads busy.

## `--drop-local` clears untracked weft files the target tracks

`DropLocal` archived a weft's local tip.
Its `reset --keep` then refused because an untracked `_lym/reviews/code/generation-record.json` sat where origin's tip has its own copy.
Before it moves a records branch, `DropLocal` now removes each untracked, unignored file of that worktree that the origin tip tracks.
It does this after the archive tag and before the reset.
Each removal goes through the destructive-path gate under a new ownership kind, `ownedUntrackedTrackedAt`.
That kind requires that the worktree's index not track the file and that the target commit track it.
Each removal is recorded as `path_removed`.
Ignored files are left alone, since git overwrites them without refusing.
A code branch's untracked files are never removed.

## A re-run of a held-back adopt judges the hub again

A record stopped at the `done` step resumed at `done`.
That step only repeats the held-back stop, so every later `lym adopt` printed the old `gone` and `needs_replay` branches, even after the pairs and run records behind them were gone.
`rejudges` now marks such a record for a restart at `reconcile` with its steps cleared, the same restart a changed `--keep-local` or `--drop-local` makes.
The recorded paused slugs are kept, so a slug an operator resumed stays resumed.
A record stopped at any other step resumes as before.
A deploy stop, for example, still resumes at `deploy`.

## Design decisions

- The pause takes the board's first Idea (exempt the adopt) over refusing it.
  An adopt is an explicit operator verb, and the gate already carries the exemption for such verbs.
- The push-lock wait is an option of the reconcile rather than a change to every reconcile.
  `lym cleanup`, hibernate, resume and the other callers keep their try-once behavior.
  It applies to the push lock of every records branch, Bolt's included, not only a weft's.
- The re-judge restarts only a record stopped at `done`.
  Earlier stops already restart at `reconcile` through `ResumeStep`, and later ones re-judge their own step.
  It needs no `lym adopt --discard` way forward, because the fresh reconcile drops the stops that no longer hold.

## Tests

- `TestRunWorker` gains a row where the exemption comes before the reconcile and is restored on return.
  It also gains a held-back record whose live pair has since ended: its re-run reconciles again and ends done.
- `TestRunAdopt` asserts that every call exempts the gate once, before any Bolt pull, and restores it once.
- `TestReconcileOrigin_Moves` gains rows where a held weft push lock is released during the wait, so the weft moves.
  In another row the lock stays held past the bound, so the weft reads busy after the expected number of polls.
- `TestDropLocal` gains a weft with an untracked file that origin's tip tracks.
  The file is removed, the weft lands on origin's tip with its content, and the mutations read tag, removal, drop.
- `TestGate_AbsentTargetIsNoOp` covers the new ownership kind.
