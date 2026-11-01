# Fixer report — discussion, round 2

Every finding's premise was checked against the code at HEAD and held.
Every finding is fixed; none is disputed or deferred.

## Changes

- **F1** (BLOCKING), fixed differently.
  D6's slug is now `<prefix>-t<N>`, where `<N>` is one more than the highest `N` of the subject's prefix on the board or among `shedrun`'s run records, so no filing reuses a slug.
  The proposed try number was not used, because a hand adopt resets the try count (D5), so `-t1` would recur after a landed `-t1`.
  A first own change, a serial kept beside the holds file, was replaced after the self-check: that file is machine-local, so the serial repeats across a machine switch (fix-proposal).
  The long-subject hash rule stays, cut to leave room for `-t` and three digits.
  Added D6 criteria: a second try after a landed first try starts, and fix tasks filed on two machines or across a hand adopt get two slugs.
- **F2** (BLOCKING), fixed as proposed.
  D6's pre-filing removal now removes the subject's own fix task when it is draft or halted (run-status state `blocked` or `failed`).
  The open-front-task rule now applies only to *other* front tasks.
  D6's closing paragraph was updated so the red check also removes the halted task, and a criterion was added.
- **F3** (BLOCKING), fixed as proposed, with the self-check's corrections below.
  D8's removal runs cancel only when the slug has a run claim, record or pair, and counts cancel's nothing-left refusal as step 1 done.
  D5 and D6 no longer file over a removal that failed.
  Their notices name the hand removal D8 gives for the failed step.
- **F4** (BLOCKING), fixed as proposed.
  D8 adds the owner test, through `EffectiveOwner`/`PrimeOwner`, for dobby's record and for running an action.
  `lym adopt check <subject>` makes the same test at its start.
  A criterion was added.
- **F5** (BLOCKING), fixed as proposed (first option).
  D4 adds `holds.json` per subject under `fabricengine.AdoptScratchDir`, written through `internal/state` under its own lock.
  D5's subject-start rule, D5's green-end lift and the D4 criterion now read that file.
  Self-check change: the green end removes only the slugs it read, under the lock, and does not empty the file.
- **F6** (BLOCKING), fixed as proposed (first option).
  `lym batten cancel` is added as the third lifter in D4's bound paragraph, in `PATTERN-adopt-hold`, and in the D3 criterion.
- **F7** (MEDIUM), fixed as proposed (second option).
  D8's filing is a create, which the draft rule makes draft, followed by a release write.
  A draft left by a filing that dies between the two writes is removed by D6's or D5's removal at the next check.
  The heal of open entries now covers a death between the release and the record.
- **F8** (MEDIUM), fixed as proposed.
  D9 adds a second marker file.
  For the body's stop, the darn producer puts the report's own way forward in the halt reason with no `darnStopWayForward`.
  The criterion was extended.
- **F9** (MEDIUM), fixed differently in one detail.
  D7 adds `ended` before `running_here`, plus its criterion and an auto-mode assumption.
  The proposed status `abandoned` does not exist on the board: `boardengine` declares only `StatusDraft` and run statuses.
  So `ended` reads any status that is neither empty, a run status nor done, draft included.
- **F10** (MEDIUM), fixed as proposed.
  D7 defines the runs as the union of `movedRuns` before and after the reconcile, the held released tasks and the open front task.
  An entry with no run record reports no row.
  A held released task reads `held_failed_check`/`waiting_for_check`, and an unstarted fix task reads `running_here` with no row.
- **F11** (MEDIUM), fixed as proposed (first option).
  D5's green end removes a draft, released, running or halted fix task, its cancel ending a running one.
  The criterion was updated to match, and the D4 criterion now says the green end may remove a released fix task first.
- **F12** (MEDIUM), fixed differently in its way forward.
  D8 refuses a change of project or kind of a front task on the payload paths, and refuses `lym board rename` of one.
  The bound sentence, `PATTERN-refusal-way-forward`, `PATTERN-front-task` and a criterion follow.
  The refusal does not name "fronttask's removal", which no command reaches.
  It names `lym board remove <slug>`, preceded by `lym batten cancel <slug> --remove` while a run holds the entry.
  It also says that the hand removal runs the `on_land` action, which re-checks the subject, so the re-run of `lym adopt` is not named.
- **F13** (LOW), fixed as proposed: `halted` reads a run record state of `blocked` or `failed` (`internal/shedengine/status.go`).
- **F14** (LOW), fixed as proposed.
  The `notify_on_land` clause was cut from D8 and added to `support-log.md`'s Rejected alternatives.

## Self-check

| Lens | Severity | Finding | Disposition |
|---|---|---|---|
| fix-proposal | BLOCKING | the fix-filing serial is machine-local, so `-t<N>` repeats after a machine switch | fixed with own change: `<N>` derived from the board and `shedrun`'s run records, which reach every machine through the reconcile (the fork's second way out) |
| fix-proposal | MEDIUM | the green end emptying the holds file drops a slug a second adopt appends during the lift | fixed as proposed: remove only the slugs read, under the lock (D4, D5) |
| fix-claims, fix-proposal (merged) | MEDIUM | `lym board remove` named for every failed removal step, but the run lock refuses it while the cancel failed | fixed as proposed: D8 names the hand removal per step, `lym batten cancel <slug> --remove` first when the cancel failed; D5, D6 and the D6 criterion point at it |
| fix-way-forward | MEDIUM | the front-task refusal names cancel first, which refuses nothing-left for a never-started task | fixed as proposed: the refusal names `lym board remove <slug>`, with cancel only while a run holds the entry; a criterion was added |
| fix-way-forward, fix-consistency (merged) | LOW | a hand removal over a standing record runs `adopt-check`, so naming a re-run of `lym adopt` doubles the check | fixed as proposed: the D6 notice names the re-run only when the failed step came after the record's drop; the D8 refusal no longer names a re-run |
| fix-claims | LOW | cancel's origin-unreachable refusal is not the nothing-left refusal | fixed as proposed: D8 says any other cancel refusal fails the step |
| fix-proposal | LOW | the draft left by a dying filing waits for a check nothing starts, and no notice says so | fixed as proposed: D8 says the dead check shows in `lym beam status` and the status-bar mark (`checkView`), and names the operator's re-run of `lym adopt <subject>` |
| fix-consistency | LOW | D3 still says "files the same darn again" | fixed as proposed: D3 adds "under a new slug, D6" |
| fix-consistency | LOW | `ended` and the fixer's own choices have no auto-mode assumption lines | fixed as proposed: lines added for `ended`, the holds file, the slug number, create-then-release and the second marker |
| fix-consistency | LOW | `PATTERN-front-task` lacks the owner bound and the fixed routing | fixed as proposed |
| fix-consistency | LOW | the D4 criterion "dobby starts it" contradicts D5's green-end removal of a released fix task | fixed as proposed: the criterion carves out a green end that comes first |

Every fork returned a report.

## Gate repair

The gate found two PATTERN changes over the 280-character cap of a rule text: `PATTERN-adopt-hold` (338) and `PATTERN-front-task` (386).
Both were reworded within the cap, keeping every rule they carried, the cancel lifter (F6), the owner bound (F4) and the fixed routing (F12) included.

## Disputed

None.

## Deferred

None.
