---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/03-driver-notify-rule.md, _lyx/plan/04-shuttle-agent-activity.md, _lyx/plan/05-run-shed-waits.md
    summary: Cards 3, 4 and 5 cite the wrong sibling card (card 8 writes the watched marker, card 7 reads the activity, card 6 replaces noticeStep); the real owners are cards 6, 8 and 7
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/03-driver-notify-rule.md, _lyx/plan/06-batten-watched-marker.md
    summary: Neither card that suppresses the driver's run-stopped message states in its Intent what the suppression can hide (stops unannounced once batten or the orch strand is gone after the render) and what bounds it
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/03-driver-notify-rule.md
    summary: Moving the notify rule out of the driver stencil takes it out of the recipe-blindness scan, and the watched rule as written ("batten announces those stops") would name a recipe the scan forbids
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/01-orch-compact-boundary.md
    summary: Three of card 1's four planned test cases restate behavior that existing compact tests already pin; only the post-boundary turn end and the at-entry flip are new
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/05-run-shed-waits.md
    summary: Card 5 plans arm-event tests for behavior existing arm tests cover and does not name the test whose per-poll fold premise D3 removes
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/plan/07-state-change-notices.md, _lyx/plan/08-activity-notices.md
    summary: Cards 7 and 8 plan wire and quiet-notice tests beside existing tests that already cover the Notify wiring, the no-strand case and the quiet conditions
  - id: F7
    severity: NIT
    class: consistency
    location: _lyx/plan/03-driver-notify-rule.md, _lyx/plan/10-reed-window-op.md
    summary: A Uses entry on each card names a file the Intent never reads from (proc_linux.go only, windowtarget.go)
---

### [MEDIUM] Cross-references to sibling cards are off by one or two

**Location:** `03-driver-notify-rule.md` (Intent, second paragraph), `04-shuttle-agent-activity.md` (Intent first line and ImpactSummary), `05-run-shed-waits.md` (Intent, per-check paragraph and Bound paragraph).
**Issue:**
Card 3 says of the watched marker "card 8 writes it"; card 6 (`batten-watched-marker`) writes it, and card 8 is the activity notices.
Card 4 says its reading is "for batten's quiet and api-error notices (card 7)" and that "the new surface has no caller until card 7"; card 8 (`activity-notices`) is the caller, and card 7 is the state-change notices.
Card 5 says "Today's `noticeStep` ... (card 6 replaces it)" and "the notices (cards 6 and 7) bound it"; card 7 replaces `noticeStep` and the notices are cards 7 and 8, while card 6 only adds the marker seam.
Cards 6, 7 and 8 reference each other correctly, so the Card Index and these three cards disagree.
A card is dispatched on its own text, so an implementer of card 5 would look for the `noticeStep` replacement in a card that does not do it, and card 3's marker pid is attributed to a card that never writes it.
**Fix:** Correct the references: card 3 "card 6 writes it"; card 4 "card 8" in both places; card 5 "card 7 replaces it" and "the notices (cards 7 and 8)".

### [MEDIUM] The cards that silence the driver do not state what the silence can hide

**Location:** `03-driver-notify-rule.md` (Intent), `06-batten-watched-marker.md` (Intent, Bound paragraph).
**Issue:**
Card 3 introduces the change that makes a watched driver send no message when the run stops, halts or finishes.
Its Intent says what the Go-side render decides and that the rule is fixed at the fresh launch, but it never says what that can now let through: once the driver is launched watched, a batten that dies afterwards, or an orch strand that disappears afterwards, leaves the run's stops announced by nobody until batten runs again (decision record D6 Bound and Open risks).
Card 6 states a bound, but its closing sentence points at "(Open risks)", a section of the decision record the plan does not carry, instead of saying what is hidden.
The rubric requires the card that introduces the change to state in its own `**Intent:**` what it can skip or let through and what bounds it.
**Fix:** In card 3's Intent state that a watched-rendered driver stays silent for the rest of its life even when batten later dies or the orch strand disappears, so those stops go unannounced until a batten runs again, and that the bound is the pid-liveness test at render, the "duplicate, never a loss" direction for an unwatched driver adopted later, and the batten notices of cards 7 and 8.
In card 6 replace the "(Open risks)" pointer with that statement.

### [MEDIUM] The moved notify rule escapes the driver stencil's recipe-blindness scan

**Location:** `03-driver-notify-rule.md` (Create, Intent third paragraph); existing pin at `cmd/lyx/driverstencil_test.go` (`TestDriverStencil_IsRecipeBlind`).
**Issue:**
`TestDriverStencil_IsRecipeBlind` scans only `stencils.ShedTemplateDriver` and fails on every shipped recipe name as a whole word (`shedrun.RecipeNames()` is `batten` and `loom`) and on the literals `lyx batten` and `lyx loom`.
The driver stencil today names neither.
Card 3 moves the `## Notifying the parent` body into two new registered stencils, so the text that decides what the driver says to its parent leaves the scanned file and the scan covers none of it.
Card 3 also words the watched rule as one that sends nothing "because batten announces those stops"; if that wording lands in the stencil, it names a recipe in a prompt whose design is that the recipe is a property of the seed alone, and the scan would reject it as soon as it covered the new stencils.
Card 3 does not target `cmd/lyx/driverstencil_test.go`.
**Fix:** Add to card 3's Intent that both new stencils stay recipe-blind and say nothing of who announces a stop (for example "stops reach the parent by other means"), and add `cmd/lyx/driverstencil_test.go` to the card's Edit list to scan the two registered notify stencils with the same token list.

### [LOW] Card 1 re-plans behavior its neighbours already test

**Location:** `01-orch-compact-boundary.md` (Tests).
**Issue:**
The four planned cases largely duplicate existing tests in `watcher_compact_test.go`:
`TestCompact_CompletesOnBoundaryAfterEntryAndIdle` already asserts that a boundary after entry completes with the idle probe (reload starts, `CycleCount` 1, `CompactionBaseline` the boundary, no abort) and that a boundary before entry does not complete;
`TestCompact_RestartWithBoundaryReadTypesNothing` already asserts that a restarted watcher with a read boundary completes without typing `/compact` again;
`TestCompact_TimeoutReturnsToIdleAndHoldsAutomaticTriggers` already asserts the timeout abort.
What is new is the boundary followed by an assistant turn end (a non-compacted `ContextTokens` reading), the boundary with no turn end after it, the at-entry flip from completing to not completing, and the migration of `landBoundary` from the fake's `boundary` map to its `autoCompact` map, which every existing compacting test using `landBoundary` depends on.
The card says "extending the existing compacting tests" but lists the cases as if all were new, and does not name the migration.
**Fix:** Rewrite the Tests list as the delta: migrate `landBoundary` to `autoCompact` (naming that the existing compacting tests follow it), add a row for a post-boundary turn end and one for no turn end after the boundary into `TestCompact_CompletesOnBoundaryAfterEntryAndIdle` and `TestCompact_RestartWithBoundaryReadTypesNothing`, and keep the at-entry rewrite.

### [LOW] Card 5 does not separate the rewritten arm tests from the new ones

**Location:** `05-run-shed-waits.md` (Tests).
**Issue:**
Several planned cases are behavior the existing arm tests already pin, and only their return shape changes:
the awaiting decision resume is `TestInnerRun_AwaitingResumesOncePerApproval` and `TestInnerRun_AwaitingResumesOnARejectionOnceAndAgainOnASameSecondApproval`;
the not-parked retry is `TestInnerRun_AwaitingNotParkedRetriesWithoutMarkerThenResumes`;
the done child returning `Done` on driver end or grace is `TestInnerRun_DoneWaitsForLiveDriverThenFinishes` and `TestInnerRun_DoneFinishesPastGraceWithLiveDriverAndWarns`.
`TestInnerRun_AwaitingPollsFoldIntoOneHistoryEntry` pins the per-poll fold that D3 removes, so it has to be rewritten to the new premise, and the card never names it.
The card says "extending the existing table and arm tests rather than adding parallel ones" without saying which.
**Fix:** Name those tests as the ones extended or rewritten, add only the cases with no existing equivalent as rows (in-call state change, baseline stat race, modification-time-unchanged decode, pause, failed stat), and name `TestInnerRun_AwaitingPollsFoldIntoOneHistoryEntry` as rewritten.

### [LOW] Cards 7 and 8 plan tests beside existing ones that cover the same behavior

**Location:** `07-state-change-notices.md` (Tests, last two bullets and the restart and retry bullet), `08-activity-notices.md` (Tests, first bullet).
**Issue:**
`wire_seams_test.go` already has `TestWire_NotifyQueuesOnThePrimesOrch` and `TestWire_NotifyWritesNothingWithoutAnOrchStrand`, which cover the `Notify` wiring and the no-strand case; card 7's "`Notify` reports `QueueNotice`'s answer" belongs as an assertion in those.
In `notice_test.go`, `TestNotice_StateChangeNotifiesOncePerEpisode`, `TestNotice_StatusChangeStartsNewEpisode`, `TestNotice_RestartDoesNotRenotify` and `TestNotice_NotifyErrorDoesNotChangeOutcome` already cover once-per-episode, a new episode on a state change, no re-notice after restart, and an error not changing the outcome; card 7's cases for the same are rows in those, changed for the retry and the sent-only-when-queued rule.
`TestNotice_QuietOnlyForRunningChildWithLiveDriver` pins quiet by status-file modification time, which card 8 replaces by activity, so card 8's quiet cases rewrite that test and are not additions.
**Fix:** Name those tests in cards 7 and 8 as the ones extended or rewritten, and keep as new cases only the report-timed notice, the parent-notice relay rule, the strand-recorded gate, the retry cap, and the API-error notice.

### [NIT] Uses entries that the Intent never uses

**Location:** `03-driver-notify-rule.md` (Uses), `10-reed-window-op.md` (Uses).
**Issue:**
Card 3 lists `internal/proc/proc_linux.go` under Uses, but the Intent calls `proc.IsAlive`, which has a Windows twin in `proc_windows.go`, so the platform file is the wrong unit.
Card 10 lists `internal/reedengine/windowtarget.go`, but the Intent never says what the window op reads from the addressing seam.
**Fix:** In card 3 drop the `proc_linux.go` entry or name the package-level `proc.IsAlive`; in card 10 either state what the op takes from `windowtarget.go` or drop the entry.

## Checked, no finding

- **Fidelity:** D1 through D7 and every Constraint in the decision record are carried: D1 card 1, D2 card 2, D3 card 5, D4 card 7, D5 cards 4 and 8, D6 cards 3 and 6, D7 cards 9, 10 and 11.
  Cards 7 and 8 introduce no scope beyond D3, D4 and D5: the retry cap, the constants, the last send attempt and the `OrchStrandRecorded` seam are all in the record.
- **Attack surface:** Cards 5, 7, 8, 10 and 11 state their bounds in the Intent (no time limit and the pause, cancellation and notices that bound it; the 3-minute and 10-minute waits; the informational limits; the window living as long as the reed session and the lock refusal read from batten's log).
- **Granularity:** cards 1 and 2 and cards 9 and 10 are each independently reviewable.
  Card 2's compact half rides on card 1's completion branch but is a separate decision; card 9 is a refactor of existing behavior and card 10 a new op.
- **Verify:** `## verify:` follows the template order and `./...` covers every package a card targets, `contracts` included, as the repository is one module.
- **ImpactSummary:** each Edit card's conclusion states a blast radius (callers, signature changes, fields) and not its Intent.
- **Test economy for card 4:** `ReadAgentActivity` and `ActivityReader` are new surface; `ReadWaitMarker`'s tests cover a different function, so no existing test covers the planned cases.
