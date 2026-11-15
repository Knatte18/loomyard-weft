# Fixer report — discussion round 3

Every finding's premise held against the record and the code
(`internal/landingshed/finalize.go`, `internal/fabricengine/replay.go`, `internal/fabricengine/pushanchored.go`,
`internal/battencli/cancel.go`, `internal/pauserecord/hold.go`).
Files changed: `_lym/discussion/decision-record.md` and `_lym/discussion/support-log.md`.

## Changes

- F1, fixed as proposed, option (a).
  D1 now defines past the parent-side merge as origin's fetched parent branch holding the task branch's work, never the local parent branch, and names that as an act-rule input.
  D2 has the Finalize push the parent branch before its transition's records and before the board `done`.
  D4 orders the Finalize: owner read, local merge, parent push, transition push, board `done`.
  It covers a takeover between the local merge and the parent push: the adopter merges again, and the old owner's hand-over archives and resets its local parent branch.
  It also covers a parent push rejected over a parent that already holds the work.
  D6 step 4 and the `PATTERN-owner-machine` constraint follow.
  New acceptance lines: D1 local-versus-origin parent, D4 old owner merged and taken over before its parent push, D4 board `done` only after the parent push.
- F2, fixed as proposed.
  `--release` is gone from D7, D8, the Constraints and the acceptance criteria.
  D8 adds `lym adopt --take-hold <subject>...`: it refuses unless the standing hold names the same subjects, rewrites the hold to name this machine, then runs steps 2 to 8.
  The hold clears only at that adopt's green check or the fix task's green re-check.
  D8 states its bound and the shell-guard deny, and says the stopped adopter stops at its next hold re-read.
  D7's hold now carries its adopt's subjects.
  D4's takeover-retry stop, the step 8 fix-try limit and the D4, D7 and D8 acceptance lines are rewritten.
- F3, fixed as proposed.
  D10 is cut to the parent's decision: the session id stays in `orchengine.State`, no tracked `_lym/orch` record exists, and no transcript leaves its machine.
  D9's last bullet launches fresh on a machine with no local transcript.
  D8 step 6, the Scope Out pointer, the Constraints clauses on the mirror and transcript ref, the mirror interval and chunk size, and the D10 acceptance lines are gone; one D10 line remains.
  The support log's three mirror rejected alternatives are now one line.
- F4, fixed as proposed: Open risks reads "None."
- F5, fixed as proposed.
  D2 puts the order in `fabricengine.PushPairAnchored` for every caller: records side first, code side only once the records side landed, none when it fails or is rejected.
  D4 hands over on a rejected records push from any caller.
  New acceptance line: D4 old owner's `lym loom commit-records` after a takeover.
- F6, fixed as proposed.
  D3 names every path that removes a cancelled run's pair as deleting its batten record in the same step, and `lym batten cleanup` as the repair only.
  D11 deletes the record in the same step.
  D1 reads a leftover `cancelled` record as no record.
  The D3 acceptance lines are split into keep and remove, and the new remove line has the dobby start the released task again.
- F7, fixed as proposed for the admission; the hold part was refined in the self-check (P2).
  D2 turns a teardown-lock wait past `fabricengine.StatusPushLockWait` into an owed push, which releases the admission.
  D7 writes the upgrade hold only when an entry names subjects to re-run.
  New D2 acceptance line; the D7 acceptance line is extended.
- Support log: a Round 3 section records these changes.

## Self-check

- fix-consistency, MEDIUM, D8 step 1 refused every other machine's hold with no `--take-hold` exception: fixed as proposed.
- fix-consistency, LOW, D8 step 8 filed a second fix task under a taken `adopt-fix` hold: fixed as proposed.
- fix-consistency, LOW, D1's act-rule inputs did not name the fetched parent branch: fixed as proposed.
- fix-proposal, MEDIUM, push skip left a landing never past its merge and never `done`: fixed with own change.
  D1 and D4 now say that under the push skip the local parent branch decides past-merge and the board `done` follows the local merge.
- fix-proposal, MEDIUM, F7 dropped the upgrade hold over a released queued task's entry: fixed as proposed.
  The upgrade hold is written when an entry became a pause intent or belonged to a released `queued` task.
  Only an upgrade whose entries all belong to landed runs or runs owned elsewhere writes none.
- fix-proposal, LOW, the cancelled record's pair-gone test read one machine's local pair: fixed as proposed.
  D1 now reads the task branch gone from origin, and a kept cancel reads `cancelled` everywhere.
- fix-proposal, LOW, a withheld code side had no way forward: fixed with own change.
  D2 says the withheld code side is reported with the records side's failure and sent by the caller's next pair push.
- fix-way-forward, LOW, `--take-hold` with no standing hold had no way forward: fixed as proposed.
  With no hold, or this machine's own hold, it runs as `lym adopt` does.
- fix-way-forward, LOW, a stopped adopter whose hold is gone named no machine: fixed as proposed.
  D4 and D8 name the subjects' owner read from the fetched Bolt.
- fix-claims: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None reached this round.
