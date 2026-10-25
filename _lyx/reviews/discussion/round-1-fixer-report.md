# Fixer report — discussion, round 1

Every finding's premise held against the code (`internal/runproc/end.go`, `internal/beamcli/dobbydeps.go`, `internal/fabricengine/spawn.go`, `remove.go`, `reconcile.go`, `recordsprobe.go`, `clone.go`, `internal/lyxcwd/anchor.go`, `internal/dobbyengine/start.go`, the three `EnsureLive` callers).
Every finding is fixed, in `_lyx/discussion/decision-record.md`, with the deliberation in `_lyx/discussion/support-log.md`.
`lyx loom validate-discussion` reports `ok`.

## Changes

- F1 — fixed differently.
  Every D7 refusal now runs before anything is touched.
  The dobbies end through `runproc.EndSparing`, sparing the push and sync roles outside their process trees, and the record says `End` takes descendants with it.
  Not made: the proposed refusal of a pass in flight.
  A live dobby keeps starting continue passes, a pause included (`internal/dobbyengine/doc.go`: "While a pause covers the run, only continue passes run"), so that refusal could fire on every attempt.
  Instead, while the move record stands every covered dobby starts no pass and no task, and the verb waits for passes in flight before it ends the dobbies.
  The criterion about a pass in flight finishing is rewritten.
- F2 — fixed differently.
  D7 now states that a change commits `B` on the shared weft branch.
  D10 stops the pull and push on other machines, and its way forward is the integrating `lyx fabric anchor <slug> --set <anchor>`, which exists for the prime and pairs alike.
  The integration is defined in D10: replay the local commits re-rooted to the target anchor, then move.
  The owner refusal exempts taking up the upstream's anchor, and a criterion covers the second machine's prime.
  Not made: having the prime pull run D7's steps itself.
  A pull runs from live sessions, which D7's refusals forbid, and the push side must stop too.
- F3 — fixed differently.
  Recovery reads a move record in the weft's lock directory `.weft/`, holding `A`, `B`, the kind and the pre-integration tip, plus a move lock held by the live run.
  It does not read the trailer or a fork-point test.
  The record is untracked and per checkout, so a child never inherits it.
  Landing is read from `HEAD`'s marker for a move, and from `HEAD` against the recorded tip for an integration.
  The trailer is dropped, the no-op rule is defined, and the criterion names both kill points and the forked child.
- F4 — fixed as proposed.
  Reconcile refuses a pair with a standing move record, naming a change in progress or the re-run.
  The reconcile sentence in D7 now states what the wiring repair does, and the third plan-writer note is dropped.
- F5 — fixed as proposed.
  The undo resets the move's pathspecs to `HEAD`, and the dirtiness check runs after it, on the rest of the weft.
  The rules for where `.lyx` sits cover each existing combination, with a by-hand way forward when both exist.
- F6 — fixed as proposed.
  Step 8 starts the worktree's dobby when its lock reads free and this machine owns the beam; for the prime it runs `dobbyengine.EnsureLive`.
- F7 — fixed as proposed.
  `EnsureLive` starts no covered dobby while the record stands, and reports why.
  The bound now names only hand starts.
- F8 — fixed differently.
  A child pass in flight and a held parent lock are waited out rather than refused, for the reason given under F1, and `lyx tender pause` is not named.
  The refusal criterion lists the wait.
- F9 — fixed as proposed, and extended.
  The pull stops only when the tip it would leave checked out carries another marker, or when it would replay commits across a marker change; a local-ahead weft is not stopped.
  The extension: the replay-across case also stops, so that upstream commits made at the old anchor are not stranded.
- F10 — fixed as proposed.
  D8 spawns the weft push for each marker commit, and the census completes on a later reconcile.
- F11 — fixed as proposed, with the third option.
  An Open risk covers unfetched branches and lagging machines, with the operator step of running reconcile first on each updated machine.
  D8 keeps the census limit explicit.
  `lyxcwd` has no notion of the prime, so a fallback through it was rejected.
- F12 — fixed as proposed.
  An empty committed marker counts as none in the census.
- F13 — fixed as proposed.
  D8 removes a leftover `.fabric-anchor`, and D4 runs clone's stale guard only on the Bolt fallback path.
- F14 — fixed as proposed.
  `probeWeftBinding` also reads `main-weft` by name, and the bootstrap guard counts a marker on either branch.
- F15 — fixed as proposed.
  All refusals precede step 1.
- F16 — fixed as proposed.
  D6's re-run recovers through the move record, and a skip on an equal anchor finishes the create steps.

## Self-check

- fix-consistency, MEDIUM: an integration on a checkout already at `B` cannot be told landed from `HEAD`'s marker. Fixed as proposed: the record holds the pre-integration tip.
- fix-consistency, LOW: the empty-marker criterion omits push. Fixed as proposed.
- fix-consistency, LOW: the D10 rationale does not cover the integration. Fixed as proposed.
- fix-consistency, LOW: the support log's Turn 4 lines contradict the new D7. Fixed as proposed: those lines are marked superseded.
- fix-proposal, MEDIUM: the `lyx tender pause` way forward does not stop continue passes. Fixed differently: the dobby honours the move record and the verb waits (see F1 and F8).
- fix-way-forward, MEDIUM: the integration conflict's way forward makes no progress. Fixed differently: the record names the conflicting commit and a hand `git reset --keep` to its parent.
- fix-way-forward, MEDIUM: a standing record does not tell a live run from a killed one. Fixed as proposed: the move lock is held by the live run, refused on a re-run and reported by reconcile.
- fix-way-forward, MEDIUM: the `EnsureLive` skip names no way forward. Fixed as proposed: it reports a change in progress, or the re-run.
- fix-way-forward, LOW: the child is left paused. Fixed differently: the pause way forward is dropped (see the fix-proposal finding).
- fix-way-forward, LOW: an integrating run on a warp that lacks `B` is refused by validation. Fixed as proposed: the refusal names a hand warp update.
- fix-claims, MEDIUM: `EndSparing` does not spare a push inside an ended dobby's process tree. Fixed as proposed: the text now spares only pushes outside those trees, and says the next push carries the commits of one that was ended.
- fix-claims, LOW: step 8 ignores beam ownership. Fixed as proposed: it starts only a beam this machine owns.

Every fork returned a report.

## Disputed

None.

## Deferred

None.
