# A loom run checks origin and its owner before each push

A run that keeps going after the machine wakes, or after the operator worked on another machine, used to push on a stale base. Nothing told it until a push was refused or a landing conflicted.
Landing already judged origin and the owner inside Parent-Align and Finalize, but the pushes in between had no check.
These are the per-transition status push, `lym loom commit-records` and PR-Review's push of the task branch.
A pass in flight across a sleep never ran the full-pass reconcile dobby owes after a wake, so it kept pushing.

## What changes

Every push of a loom run's pair now passes a pre-push check first (`internal/loomcli/prepush.go`).
The check runs the same run-scope origin reconcile the landing rows run (`runReconciler`).
That reconcile fetches what origin moved, fast-forwards or replays the run's own branches where it can, and reads the owner of the run, its beam and the root from origin's Bolt.
It then acts on the run's own entry:

- **Owned by another machine:** the run is paused under the owner caller through `tendercli.PauseOwnedElsewhere`, and the push does not run.
  The stop names `lym adopt <subject>`, using the beam or `root` the owner verdict carries.
- **Stopped by a branch** (rewound, rewritten, diverged, gone, dirty and the like): the run gets a plain pair pause through the new `tendercli.PausePair`.
  The stop names each of the run's branches that stopped, the reconcile's way forward, and `lym tender resume <slug>`.
- **Ended on origin** while the parent lacks the run's work: the run is paused, and the stop names `lym cleanup --apply`.
  A run whose own landing already put its work on the parent pushes as before.
- **Active:** the push goes. When the reconcile moved the run's own branches to origin's new tips, the orch gets one line naming them.

Each stop posts one notice line to the nearest live orch.
The check is skipped when pushes are off, in the prime, while a pause covers the run, and while the run stands at Finalize.
A pause needs no check because its network gate already holds the push, and the first push after the pause lifts runs the check again.
Finalize needs none because it judges origin and owners itself and must finish once it is past its parent-side merge.
An unreachable origin or a failed check lets the push go, so the push's own failure is recorded as before.

How each push reports a stop:

- A stopped status push starts no detached push and leaves its commits local.
- `commit-records` answers an ok envelope with `push_stopped`.
- PR-Review's push halts on the stop the way it halts on any push refusal, with the stop's text as the reason.

## Design decisions

- **The check sits at the push, not at the wake,** as the entry asks.
  It covers a plain machine sleep as well as `lym hibernate request`.
  It also covers a move made while this machine was awake but idle.
- **It reuses the landing's run-scope reconcile instead of keeping a new "last seen" record.**
  The tracking refs and the local Bolt already hold what the run last saw.
  The reconcile's verdicts are what Parent-Align and Finalize act on, and its fast-forwards are how a landing takes a new base.
- **The parent branch is not moved here.** The run scope judges the parent but never moves it.
  A parent that moved on origin is taken in by the landing's Parent-Align, as before.
  The "moved" notice names only the run's own branches, so a parent behind origin does not re-notice on every transition.
- **A stop pauses the run instead of failing the status seam.** A seam error would halt the run with a commit error.
  A pause holds the driver through the existing tender machinery, matches the stops the wake reconcile already makes, and batten's owner check picks it up again when the pause lifts.
- **One grace for both pauses:** `ownerPauseGrace` became `pauseGrace` in `tendercli`, shared by `PauseOwnedElsewhere` and `PausePair`.
- **The local status commit still comes before the check;** only the push waits on it.
  A local commit does no harm, and the next push after the stop carries it.

## Bounds

- The check reads origin once per push, so a move that lands between the check and the push is caught only by the push's refusal.
- The check adds the reconcile's two `ls-remote` calls to each status push that committed something.
- Batten's own status push from the prime pushes records only and is not checked.

## Docs

The `loomcli`, `tendercli` and `battenshed` package docs describe the check, the plain pause and the narrowed bound.
`PATTERN.md`'s `PATTERN-owner-machine` entry and its background name pushing alongside starting, resuming and landing.
