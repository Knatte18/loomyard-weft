# Round 3 fixer-report — discussion

## Changes

- **F1, fixed differently.**
  - D7's first bullet now exempts both owner checks for a Finalize whose local parent branch already holds the task's work, such as the re-step after a transient parent-push failure.
    The test is `parentHoldsTaskWork` in `internal/fabricengine`, exported for Finalize and InnerRun.
    A read failure halts Finalize naming a re-run.
  - D6's carve-out names the same test, applied while Finalize runs; InnerRun keeps the lock on a read failure until the next check.
  - Acceptance criteria were added: a re-entered Finalize finishes its push, and a Finalize resumed after an owner halt before its merge meets both checks again.
    The halt/land criterion now names the exemption.
  - Not made as proposed: using `finalizeEntered` as the detector.
    It returns true on any Finalize history row (`internal/fabricengine/replay.go:259`, `historyNames` at replay.go:208-214).
    A stuck or rerouted Finalize appends such a row (`internal/shedengine/reroute.go:45,55`).
    So a Finalize halted at D7's parent-owner check and resumed here would skip both checks and merge into a parent this machine does not own.
    Its unreadable-counts-as-reached rule would also skip the guard on a doubt.
- **F2, fixed as proposed, option (a).**
  - D5 step 1 now states the stock seam's behaviour: `HubSeamOptions.HubPaused` uses `Record.Covering`, which matches slug-scope entries, and `LivePair` reads a covered pair as not live.
    Adopt builds its own seams, whose `HubPaused` covers hub entries only.
  - The adopt-back criterion's second sentence now names those seams.
  - Open risks records that a pair under a hub pause still reads not live there, so `--drop-local` can move its branches, as any hub-paused pair's do today.
- **F3, partly fixed and partly disputed.**
  - The halt note now names the reset only when origin's parent tip, fetched at the stop, holds the task's work.
    When it does not, or the fetch fails, the note keeps today's reason and way forward.
    The fresh fetch is needed because push classification reads the upstream only as last fetched (`internal/fabricengine/pushclassified.go:241`).
  - D6 now says the records branch needs no repair: the merge makes no records commit, and step 2b's records commit is replayed by the next moving reconcile like any local records commit.
  - The matching criterion was updated.
- **F4, fixed as proposed.**
  - The `PATTERN-owner-machine` entry now says the owner verdict for a resume, wake or landing is judged inside `ReconcileOrigin`.
    It names the exceptions: a start reads the Bolt it just pulled, the post-push re-read reads the Bolt it just pushed, the landing reads the parent owner from the Bolt its reconcile left, and a resume with origin unreachable reads the local Bolt.
  - The Constraints bullet names `PATTERN-owner-machine` beside `PATTERN-origin-reconcile`.

## Self-check

- fix-consistency, MEDIUM, a weft reset with no merge commit to drop: fixed as proposed by removing it (same as fix-claims BLOCKING, below).
- fix-consistency, LOW, the halt/land criterion still contradicted the exemption: fixed as proposed.
- fix-consistency, LOW, the PATTERN entry left out the landing's parent-owner read: fixed as proposed (same as fix-proposal LOW).
- fix-consistency, LOW, the carve-out's detector stays true after Finalize stops: fixed as proposed; D6 now applies the test while Finalize runs.
- fix-claims, BLOCKING, the weft reset rests on a false premise and drops unrelated records commits: fixed as proposed by removing the weft reset (`internal/fabricengine/doc.go:1203`, `merge.go:322-325`).
- fix-proposal, BLOCKING, `finalizeEntered` reads a halted or rerouted Finalize as entered: fixed as proposed by switching to `parentHoldsTaskWork`, with a read failure halting.
- fix-proposal, BLOCKING, the weft reset: duplicate of fix-claims BLOCKING.
- fix-proposal, LOW, the PATTERN entry: duplicate of fix-consistency LOW.
- fix-way-forward, MEDIUM, the halt-note condition read a stale origin: fixed as proposed by requiring a fresh fetch at the stop, with a failed fetch keeping today's note.
- fix-way-forward, LOW, `--drop-local` under a hub pause: fixed as proposed by recording it in Open risks.

## Disputed

- **F3, weft-side step, premise false.**
  The review says the parent-side merge concludes a commit on the weft side.
  The code says otherwise:
  - `internal/fabricengine/doc.go:1203`: "The weft is never a merge participant, in either verb or either direction."
  - `internal/fabricengine/merge.go:322-325`: Merge merges the warp checkout and "the weft side is not a merge participant".
  - `doc.go:88`: "`main-weft` does not advance on landing by design."

  A `git reset --keep origin/<parent-branch>-weft` would only drop unpushed records commits, including step 2b's sibling-run records.
  So D6 instead states that the records branch needs no repair.

## Deferred

None.
