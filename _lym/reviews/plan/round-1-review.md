---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/09-adopt-foreground.md:185-186,199 and _lym/plan/08-adopt-check-ends-on-hold.md:70-73
    summary: No hold re-read follows the check, so an adopter whose hold another machine took during the gate run still clears nothing yet brings every subject up, against D6's re-read before each step from step 2 on and its no-dobby criterion
    fix: In card 9's Intent, steps 7 and 8, run holdStands once more after HoldCheck returns and before runBeamSteps; a hold gone or naming another machine ends the adopt naming that machine, with no runBeamSteps call. Drop the sentence that no re-read follows steps 7 and 8. In card 8's Intent, runHoldCheck step 3, a ClearHold that reports false (another machine's hold stands) ends the result not green, naming that machine, with no BeamUp and no DropBoltHold; in step 4, a FixHold refusal over another machine's hold ends the result naming that machine, with no BeamUp. Add TestRunHoldCheck rows AHoldTakenOverDuringAGreenCheckBringsNothingUp and AHoldTakenOverDuringAFailingCheckBringsNothingUp, and a TestAdopt row AHoldTakenOverDuringTheCheckStartsNoDobby (no BeamUp call).
  - id: F2
    severity: BLOCKING
    class: consistency
    origin: handler
    location: _lym/plan/09-adopt-foreground.md:189-191
    summary: Card 9 exempts the hold's FixTask from the moved runs unconditionally, which D6 step 2 does not license, so after --take-hold of an adopt-fix hold the fix task's run is never taken over and is stranded
    fix: In card 9's Intent, step 2, exempt the hold's FixTask only when the hold read as this machine's before the adopt (WriteAdopt's Kept, not a take-hold rewrite). Under a take-hold rewrite, the fix task's run is moved unless its origin record holds a takeover entry naming this machine. Have card 2's AdoptWrite report the rewrite (a TookOver field) so runForeground can tell the two cases apart. Add a TestAdopt row TakeHoldOfAnAdoptFixHoldMovesTheFixTasksRun (AddPair and TakeOver of the fix task's run), beside the existing own-re-run row that leaves it untouched.
  - id: F3
    severity: BLOCKING
    class: scope
    origin: lens:repo-invariants
    location: _lym/plan/09-adopt-foreground.md:50-142
    summary: Card 9 deletes resumeSlug, parentWarning's only caller, and lists parentWarning nowhere, so the PATTERN-test-only-code guard reports it as reached by no main package; workerOutName is left unused too
    fix: Add `internal/adoptcli#parentWarning` and `internal/adoptcli#workerOutName` to card 9's Delete list.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:repo-invariants
    location: _lym/plan/09-adopt-foreground.md:44 and _lym/plan/10-retire-adopt-record.md:3-9
    summary: Card 9 targets cmd/lym, so its tier-1 batch gate runs the test-only guard while internal/adoptrecord has no importer and is not yet deleted; it passes only if Webster batches card 10 with card 9, contrary to the overview's Shared Decision
    fix: Remove card 10 with `lym loom remove-card` and add its six Delete targets (internal/adoptrecord/record.go, watch.go, doc.go, record_test.go, watch_test.go, testmain_test.go) to card 9's Delete list, with card 10's Intent sentences folded into card 9's Intent. Reword the overview's first Shared Decision to say the guard first runs at card 9's batch gate.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/02-board-adopt-write.md:51-55
    summary: FixHold and ClearHold judge another machine's hold only over the pulled store; a rejected push whose replay card 1 refuses is only logged and the push then reads level, so both report success for a change origin never took
    fix: In card 2's Intent, FixHold and ClearHold re-read the local Bolt after the push, as WriteAdopt does. A hold that then names another machine makes FixHold return the *HoldRefusal naming it and makes ClearHold report false. Give both a before-push hook, as card 3's takeoverBeforePushHook does, and add TestWriteAdopt rows FixHoldRejectedOverATakeHoldRefusesNamingTheOtherMachine and ClearHoldRejectedOverATakeHoldReportsFalse.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/03-fabric-takeover-writer.md:38-45
    summary: TakeOverRun commits the takeover entry onto the parent's records branch and its retry resets that branch to origin's tip, but only the moved pair's own tips are archived, so this machine's unpushed commits of other runs on the shared parent branch can be dropped
    fix: In card 3's Intent, archive the parent records branch's local tip origin lacks through ArchiveOwnerTips (the parent pair added to its pairs) before step 4 writes the entry, and make step 5's reset touch only the branch whose push was rejected. Add a TestTakeOverRun row AParentRecordsTipOriginLacksIsArchivedBeforeTheRetryReset.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:affected-set
    location: _lym/plan/08-adopt-check-ends-on-hold.md:60 and _lym/plan/09-adopt-foreground.md:172
    summary: The cards claim every way forward names the hold's re-run, but fixBody, fixRemovalError, removeFixTask, runCheck's owner-read and owned-elsewhere lines and fronttask's adopt-check hand step still name `lym adopt <subject>`, which card 2 refuses under a multi-subject hold
    fix: Add `internal/adoptcli#fixBody`, `internal/adoptcli#fixRemovalError`, `internal/adoptcli#removeFixTask` and `internal/fronttask#actions` to card 8's Edit list, and `internal/fronttask/doc.go` to its Uses. In card 8's Intent, those lines and runCheck's owner-read and owned-elsewhere lines name `Rerun`, the re-run over the hold's subjects; fixBody's stop condition takes the re-run from fixDeps; the adopt-check row's hand step names `lym beam status`, which shows the hold's re-run (card 6).
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/09-adopt-foreground.md:212
    summary: Card 9 rewrites the Adopt pause section of PATTERN-dobby-runs.md but leaves its opening summary line, which still says an adopt pause lifts only by hand
    fix: In card 9's Intent, add that pattern/PATTERN-dobby-runs.md's opening summary line becomes the declared PATTERN-dobby-runs text, ending in an adopt holds the hub through Bolt's holds until an adopt's check is green.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/02-board-adopt-write.md:15
    summary: holdwrite.go's file header says WriteHold is the only writer of the holds list, which card 2's three new writers falsify, and no card lists the header
    fix: In card 2's Edit list, replace `internal/boardengine#WriteHold` with `internal/boardengine/holdwrite.go`, and in its Intent say the file header names WriteHold, WriteAdopt, FixHold and ClearHold as the writers of holds.
  - id: F10
    severity: LOW
    class: scope
    origin: lens:affected-set
    location: _lym/plan/11-retired-adopt-wording.md:8-9
    summary: EndSubject's doc comment in internal/runproc/end.go still names the adopt worker among the hub's processes, and card 11 edits only runproc's doc.go and RoleAdopt
    fix: Add `internal/runproc#EndSubject` to card 11's Edit list, and in its Intent say EndSubject's doc drops the adopt worker from the processes that serve the whole hub.
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/08-adopt-check-ends-on-hold.md:87,106
    summary: Card 8 says fronttask's adopt-check row reads the exit code and would retry at every settle, but fronttask starts the action detached and reads neither its exit code nor its output
    fix: In card 8, reword the checkCmd sentence and the ImpactSummary to say fronttask's adopt-check row starts the check detached and reads neither its exit code nor its output, and drop the retry rationale from the exit rule.
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/01-board-replay-holds.md:32,42-48
    summary: The replay branch that reports a parent hold origin no longer holds as Lost has no TestFabricReplay row, though every other hold branch has one
    fix: Add a TestFabricReplay row AParentHoldOriginNoLongerHoldsIsLost to card 1, asserting origin's holds unchanged and one Lost finding.
  - id: F13
    severity: LOW
    class: design
    origin: handler
    location: _lym/plan/01-board-replay-holds.md:3-4,23
    summary: Card 1 creates parseReplayHolds to decode holds and machines from board.json bytes, which boardengine's ParseOwners already decodes into Owners.Holds and Owners.Registry
    fix: Drop card 1's Create of `internal/boardengine#parseReplayHolds`, add `internal/boardengine#ParseOwners` to its Uses, and in its Intent have Apply read the three sides' holds and registry through ParseOwners.
---

### [BLOCKING] No hold re-read between the check and the beam steps

**Location:** `_lym/plan/09-adopt-foreground.md:185-186,199`; `_lym/plan/08-adopt-check-ends-on-hold.md:70-73`

**Issue:** D6 says every adopt "re-reads its own hold from Bolt before each step from step 2 on, and stops once the hold is gone or names another machine, … starting nothing".
The acceptance criterion says such an adopter "starts no dobby".
Card 9 re-reads the hold only up to the check, then runs `runBeamSteps` "on green and on red alike".
The gate run is the longest step, so a `--take-hold` most likely lands during it.
On green, card 2's `ClearHold` reports false for another machine's hold rather than failing, and card 8's step 3 goes on to `BeamUp` and `DropBoltHold`.
On red, card 8's step 4 runs `BeamUp` after the filing even when `FixHold` refused.

### [BLOCKING] The fix task's run is never moved under `--take-hold`

**Location:** `_lym/plan/09-adopt-foreground.md:189-191`

**Issue:** D6 step 2's rule names only the hold's prior owner and the origin takeover entry.
Card 9 adds "and the hold's `FixTask` are not moved" with no condition.
That keeps an own re-run from resetting the fix task, but it also applies after a `--take-hold`.
There the fix task's run was driven by the old adopter, whose dobby hands it over once the owner changes.
The new owner never adds its pair or writes its takeover entry.
The catch-up cannot pick it up either: `HubRunSlugs` lists only runs whose pair stands here (`internal/battencli/catchup.go:280-292`).
So D6's "under `adopt-fix` the fix task stands" leaves a fix task that no machine runs.

### [BLOCKING] `parentWarning` left unreachable

**Location:** `_lym/plan/09-adopt-foreground.md:50-142`

**Issue:** `parentWarning`'s only caller is `resumeSlug` (`internal/adoptcli/adopt.go:505`), which card 9 deletes.
No card deletes or uses `parentWarning`.
`TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain` (`cmd/lym/testonlycode_linux_test.go:82-94`) then fails, breaking `PATTERN-test-only-code`.
`workerOutName` (`internal/adoptcli/caller.go:33`) also loses its last users, `spawnWorker` and `follow.go`.

### [MEDIUM] Card 9's batch gate meets an importer-less `internal/adoptrecord`

**Location:** `_lym/plan/09-adopt-foreground.md:44`; `_lym/plan/10-retire-adopt-record.md:3-9`

**Issue:** Card 9 targets `cmd/lym#unboundedWaitFiles`, which puts `cmd/lym` in its packages.
The batch gate runs tiers 0 and 1 over those packages, and the test-only guard is tier 1.
After card 9 nothing imports `internal/adoptrecord`, so the guard fails unless card 10 lands in the same batch.
The overview's Shared Decision says the guard runs only after card 9, which the verify model in `loom-plan-spec.md` does not guarantee.

### [MEDIUM] `FixHold` and `ClearHold` report success over a refused replay

**Location:** `_lym/plan/02-board-adopt-write.md:51-55`

**Issue:** A Bolt replay's `Refused` findings are only logged (`internal/fabricengine/bolt.go:393`).
`pushOwnerCommit` then finds the Bolt level and returns nil (`internal/boardengine/ownerset.go:356-376`).
`WriteAdopt` catches this with its post-push re-read, but its two sibling setters have none.
So the setters report a hold written or cleared that origin refused.

### [MEDIUM] The takeover retry can drop owed commits on the parent's records branch

**Location:** `_lym/plan/03-fabric-takeover-writer.md:38-45`

**Issue:** Step 4 commits the entry onto the parent's records branch, the prime's for a root child.
Step 5's retry resets "the records branch" with `reset --keep` to origin's new tip.
Steps 2 and 3 archive and reset only the moved pair's branches (`DropLocal` selects `target.slug == slug`, `internal/fabricengine/destroy.go:1977-1990`).
Card 9's catch-up, which pushes owed commits, runs only after the takeover.

### [MEDIUM] One-subject ways forward left under multi-subject holds

**Location:** `_lym/plan/08-adopt-check-ends-on-hold.md:60`; `_lym/plan/09-adopt-foreground.md:172`

**Issue:** The cards claim every way forward names `lym adopt <hold's subjects>`.
These still name `lym adopt <subject>`:
- `fixBody` (`internal/adoptcli/fixloop.go:190`);
- `fixRemovalError` (`fixloop.go:351`);
- `removeFixTask` (`green.go:49,57`);
- `runCheck`'s owner-read and owned-elsewhere lines (`check.go:156,159`);
- fronttask's hand step (`internal/fronttask/fronttask.go:35`).

Card 2 refuses each of these as another set under a multi-subject hold.
The refusal names the right re-run itself, so the operator is not stranded, but the claim fails.

### [LOW] `PATTERN-dobby-runs.md`'s summary line left stale

**Location:** `_lym/plan/09-adopt-foreground.md:212`

**Issue:** `pattern/PATTERN-dobby-runs.md:3` ends "an adopt pause lifts only by hand", which the declared PATTERN change replaces.
Card 9's Intent names only the section, the `--no-orch` exception and `PATTERN.md`'s line.

### [LOW] `holdwrite.go`'s header falsified

**Location:** `_lym/plan/02-board-adopt-write.md:15`

**Issue:** `internal/boardengine/holdwrite.go:1` reads "the only writer of board.json's `holds` list", which stops being true once card 2 adds its three writers.

### [LOW] `EndSubject`'s doc still names the adopt worker

**Location:** `_lym/plan/11-retired-adopt-wording.md:8-9`

**Issue:** `internal/runproc/end.go:88`, inside `EndSubject`'s span (79-93 per `lym quarry resolve`), lists "the adopt worker" among the processes that serve the whole hub.

### [LOW] Card 8 misstates how fronttask runs the check

**Location:** `_lym/plan/08-adopt-check-ends-on-hold.md:87,106`

**Issue:** `startAction` starts the action through `startDetached`, reaps it and appends its output to a file (`internal/fronttask/hub.go:111-165`).
Nothing reads the exit code.

### [LOW] Untested replay branch

**Location:** `_lym/plan/01-board-replay-holds.md:32,42-48`

**Issue:** The `Lost` branch for a parent hold origin no longer holds has no `TestFabricReplay` row.

### [LOW] `parseReplayHolds` duplicates `ParseOwners`

**Location:** `_lym/plan/01-board-replay-holds.md:3-4,23`

**Issue:** `ParseOwners` (`internal/boardengine/owners.go:61-80`) already decodes `machines` and `holds` from `board.json` bytes into `Owners.Registry` and `Owners.Holds`.
Card 1 creates a second decoder of the same keys.

The target-changing fixes (F3, F7, F9, F10, F12, F13) were overlaid together with `lym loom validate-plan --overlay` and report no blocking finding beyond the empty baseline.
F7's fix includes `internal/fronttask/doc.go` under Uses, which the first combined run required.
F4's fix removes a card, so it was not overlaid.

## Rejected

- guard-reach F5 (card 5's `UnderOwnHold` true branch unreachable): the state is reachable when card 1's replay lands the hold but reports the owner field `Lost` to origin. D6's Delivers also requires the dobby and the orch watcher to read the hold.
- guard-reach F6 (a stale fix task's other-set refusal retried at every settle): fronttask starts the action detached and never reads its exit code (`internal/fronttask/hub.go:111-165`), so a non-zero exit causes no retry.
- affected-set F2, the `internal/tenderengine/doc.go:61` part: "An adopt pause is an operator pause" stays true for the legacy `adopt` entries that pauserecord still reads. The `runproc/end.go` part is kept as F10.
- Card 2 case 5 spelling `lym adopt --take-hold` in a refusal: only a caller that passed the flag reaches it, and card 4's row denies the flag to every agent. The decision record's constraint covers refusals an agent reads, and the overview's Shared Decision covers the plain adopt's refusal, which never spells the flag.
- `orchcli.DobbyProgram` created on card 8 rather than a card of its own: it is a one-line constant moved for its first cross-package consumer, not an independently reviewable unit.
- Card 9 refusing the take-hold flag with `--all`: D6 does not name this refusal. It follows from a hold naming its subjects, the card bounds and words it, and a TestAdopt row reaches it.
