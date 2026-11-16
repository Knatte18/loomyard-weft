# Round 1 fixer report — discussion decision record

Every changed line is in `_lym/discussion/decision-record.md`.
F2 and F13 are `decision`-class, so they are with the parent and left out of this report.

## Changes

- F1, as proposed: D1 retires the pending-flag respawn (`respawnForeignFlags`, run by `coalesceFlagged`).
  A flag the holder does not owe stays on disk for its worktree's next push, which in a pair is the records-first `PushPairAnchored` with D2's takeover read.
  Added a D1 criterion: a flagged code side never reaches origin before its records side, and never after a takeover.
- F3, as proposed: D6's Bound no longer says only the operator's shell runs `--take-hold`.
  It calls the guard row a guardrail over static text that a written script passes, and bounds a bypass by no skipped check, no run before green, and the old adopter's stop at its next hold re-read.
- F4, as proposed (first option): the `--take-hold` row's steer and notice say only that taking over a hold is the operator's job, and they never spell the flag.
  The flag appears only in the row's pattern and its `Forms` entry.
  The `PATTERN-shell-guard-table` constraint and the D6 `--take-hold` criterion now match.
- F5, as proposed: under D6 step 1, a hold this machine wrote stays as it stands, with its kind, fix task and try counts.
  A re-run under its own `adopt-fix` hold files no second task.
  At the D5 upgrade, an open adopt fix task makes the hold `adopt-fix`, with the try count read from `fix-tries.json`.
  Added criteria for both.
- F6, as proposed, extended by the self-check: the owing machine pushes the tip with a lease, then reads the records again and resets with a lease when the tip was given up.
  The self-check added an owner-side guard for an owing process that dies before its re-read: a code-side push rejected over a tip no transition names resets that tip with a lease, then pushes.
  The D2 criterion now covers both cases.
- F7, as proposed: the registry is keyed by a machine id kept in `machine.json`.
  Two new refusals: a current name that is another machine's former name, and a current name another machine already holds.
  Each names `lym machine name <new>`.
  Added a criterion for both.
- F8, as proposed: D7's upgrade bullet now states the true premise, citing `internal/battencli/cleanup.go` and `internal/runfsm/state.go`.
  It gives each legacy cancelled record whose pair stands a finished cancel intent with keep.
  The criterion changed to: such a pair is listed as kept and is not torn down.
- F9, first option: a run whose push is owed gets no `bolt-hold` entry, so its owed push lands before the takeover, and the act rule still waits.
  Added a criterion.
- F10, fixed differently: a hand-over now removes the local pair through `pairteardown`'s local mode.
  It does not do this in every case, as the fix key proposed: when the caller runs inside the pair, the removal falls to the parent's dobby.
  The reason: a caller cannot remove the worktree it runs in.
  D7 gains a bullet for the dobby to remove a pair named by a hand-over mark.
  Added a criterion.
- F11, as proposed: D6 replaces `lym beam status`'s adopt section with a view of the Bolt hold, citing `internal/beamcli/cli.go`.
  Added a criterion.
- F12, as proposed: D10 writes a hub's launcher when the first dobby of a worktree owned here starts, a beam's included.
  The criterion changed to match.
- F14, as proposed: D2 names the hand-over mark, a machine-local file under `.lym` named in `internal/lymdirs`.
  A later rejection over a takeover entry a mark already names skips the hand-over and pushes nothing.
  Covered by the new D2 criterion.
- F15, as proposed: D4 step 4 pushes the archive tags and posts D2's notice; the D4 criterion now names both.
- F16, as proposed: the upgrade notice names `lym tender resume <slug>` for each run it paused, to run after the green check.
  The D5 criterion now names it.
- F17, as proposed (renamed input): `PushOwedOffline` becomes `PushOwed`, which covers an offline origin and a held teardown lock.
  Its refusal names the owed push and its last error.
  Added a criterion.
- F18, fixed differently: the proposed first option was not made, so `PastParentMerge` still reads origin's parent branch.
  Under that option, a local merge pushed under a hold could land after the adopt's check had run, so the check would never see it.
  D5's Bound now states the review's second option: such a run waits out the hold with its merge local and pushes once the hold clears.
  Its criterion matches.
- F19, as proposed: the new caller is `pauserecord.BoltHoldCaller` (new), string `bolt-hold`, distinct from the legacy `HoldCaller`.
  Every `hold` caller mention in D5, D6 and the criteria is renamed.

## Gate repair

- citation-symbol-missing at D5: `pauserecord.BoltHoldCaller` does not exist yet, so the record now marks it ` (new)`.

## Self-check

- fix-way-forward, MEDIUM, `lym machine name <new>` led to another refusal: fixed by my own change. The rename records a refused name among none of this machine's former names, and the criterion says so.
- fix-way-forward, MEDIUM, a hand-over mark would tear down a re-adopted pair: fixed by my own change. The dobby removes a pair only while another machine owns the run, and an adopt's takeover naming this machine drops the mark. Added a D7 criterion.
- fix-way-forward, MEDIUM; fix-consistency, MEDIUM; fix-proposal, MEDIUM (merged): an own-hold re-run had no subject check. Fixed as proposed: an own hold for other subjects refuses, naming them, with `lym adopt <hold's subjects>` as the way forward. Added a criterion.
- fix-way-forward, MEDIUM, the `PushOwed` refusal's way forward did not cover the lock case: fixed as proposed. The way forward is now a re-run once the owed push lands.
- fix-way-forward, LOW, the guard row's steer had no way forward: fixed as proposed. The steer names asking the operator.
- fix-consistency, MEDIUM, Scope's Out line says landed `runfsm` code is never redesigned: fixed as proposed. The line now excepts D1's input rename and refusal wording. Reverting F18 took away the other case.
- fix-consistency, LOW, D6 step 8 was not qualified for runs the upgrade paused: fixed as proposed in step 8. The fix-task criterion stays as it is, since that loop's runs carry no pause intent.
- fix-consistency, LOW, the upgrade's hold kind was stale: fixed as proposed. The first read writes "one Bolt hold, `adopt` or `adopt-fix`".
- fix-consistency, LOW, D10's line "Beams need no launcher change." clashed with the new rule: fixed as proposed. It now reads "A beam gets no launcher of its own."
- fix-proposal, MEDIUM, F6 was not crash-safe: fixed by my own change, an owner-side reset with a lease on a rejected code push, with its criterion extended (see F6 above).
- fix-proposal, MEDIUM, F18's first option let an unchecked merge land under a hold: fixed as proposed by switching to the review's second option (see F18 above).
- fix-proposal, LOW, a machine that lost `machine.json` cannot reclaim its entry: fixed as proposed in D3's Bound. Such a machine registers under a new name and adopts its subjects again.
- fix-claims: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None reached this round.
