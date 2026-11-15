---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:116
    summary: D4 detects a takeover by reading the result of a records replay, but the takeover entry and the old owner's transition edit the same batten and child records, so the replay stops on ErrReplayConflict with the tip restored and the old owner never hands over
    fix: >-
      In D4's second bullet, replace "lym replays as today and reads the run's own record from the result" with:
      the rejected push fetches the branch and reads the run's own record from origin's fetched tip before any replay;
      when that record holds a takeover entry naming another machine that this machine has not yet read, the run hands over without replaying.
      Only when it holds none does lym replay and push as today.
      Give D4's last bullet the same order for the adopter: a rejected takeover push fetches, rewrites the takeover entry onto origin's tip and pushes again,
      never replaying its own commit over a record the old owner changed.
      Keep the declared PATTERN-run-fsm text's "hands over, never replays", which then matches D4.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:180
    summary: D7's hub entry in the pause record covers every slug and the orch, so on the adopter it holds the adopt-fix task the act rule lets act, the check's and deploy's child processes, the orch D8 keeps up, and the lym batten cancel the fix loop names as way forward
    fix: >-
      Rewrite D7's third bullet: a machine that reads a hold writes no hub entry;
      it writes a slug entry, under one new hold caller, for each run of this machine the act rule answers wait for under the hold,
      and none for the adopt-fix hold's fix task.
      It drops those entries once it reads no hold.
      Add to D7 that the orch, `lym adopt`'s own process and the children it spawns for the deploy and the check, and the operator's cancel are not held by the hold's entries,
      and add to D8's Bound what the hold still lets through on those paths.
      Add an acceptance criterion under D7 that under an adopt-fix hold the fix task's agent turns and network calls pass the gate and the orch is not interrupted.
  - id: F3
    severity: BLOCKING
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:195
    summary: A hold whose adopting machine died or stays off stops every run on every machine with no stated way out, since another machine's adopt is refused over it and --release is stated only for a hold over a failing check
    fix: >-
      In D8's --release paragraph, state that `lym adopt --release` clears any standing hold, an adopt or adopt-fix one, whichever machine wrote it,
      and that the refusal of D8 step 1 over another machine's hold names the writer machine and the hold's time, with no way forward an agent reads.
      State its bound in D8's Bound: it lets the hub go on without the dead adopter's check, and only the operator's shell runs it.
      In D7's Bound, replace "A hold stands until the adopt that wrote it clears it or `lym adopt --release` (D8) clears it"
      with the same rule, naming that a hold no live adopt owns stands until the operator releases it.
      Add an acceptance criterion under D8 that a hold written by a machine that then stops is cleared by --release on another machine, whose adopt then goes on.
  - id: F4
    severity: MEDIUM
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:51
    summary: The act rule exempts a landing past its parent-side merge from hand_over only, so under a hub-wide hold it waits with its parent lock taken, indefinitely at the fix-try limit, against D6, D9 and PATTERN-owner-machine's rule that a landing in flight is never stranded
    fix: >-
      In D1's act rule, make the carve-out cover both answers: a run landing past its parent-side merge answers act under a hold and when another machine owns it,
      and list it before the hand_over and wait bullets.
      In D7's second bullet, add that a landing past its parent-side merge is not held.
      Extend the D1 acceptance criterion on landing past its parent-side merge to say act under a hold too.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:182
    summary: The pause record holds one live entry per scope, so a hold's hub entry shares the hub slot with the operator's hub-wide tender pause and hibernate; removing it can lift the operator's pause, lym tender resume lifts it under a standing hold, and every wake under a hold skips its origin reconcile
    fix: >-
      Once F2's fix moves the hold's enforcement to slug entries under their own caller, add to D7 how they coexist with the operator's hub entry and hibernate's:
      the hold's removal drops only entries of its own caller, `lym tender resume` leaves the hold's entries standing while the Bolt hold stands,
      and the hibernate wake does not read a hold caller's entry as a hub pause that skips its reconcile.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:88
    summary: D3 puts the cancel intent in the batten record that lym batten cancel removes today and records no finish, so D1's cancelled cannot be told from cancelling and D11's kept-pair rule rests on a record the cancel deletes
    fix: >-
      In D3's cancel bullet, state that a cancel keeps the batten record and writes its finish into it, the time it ended and remove or keep, committed and pushed when the cancel ends,
      in place of today's removal of the run's records.
      State in D1 that cancelled reads that finish and cancelling an intent without one.
      State in D11 how a pair cancelled with keep before the upgrade, with no batten record left, is listed rather than read as an orphan.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:90
    summary: With origin unreachable, lym batten cancel and lym tender pause and resume of one slug each push before they act, and no Decision says whether they refuse, wait or act locally, so the Goal's stop at any moment is unbounded offline
    fix: >-
      Add to D3 what each slug verb does offline.
      For example: `lym tender pause <slug>` writes this machine's pause record entry at once, so the run's processes stop, and owes the intent push under D2's retry;
      `lym batten cancel` ends the run's processes and strands at once, owes the intent push, and finishes the cancel once origin answers.
      State each path's bound: what stays only on this machine until the push lands, and that another machine reads the run unpaused or uncancelled until then.
      Add an acceptance criterion under D3 for the offline pause.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:97
    summary: D3's refusal of a non-owner's pause or cancel names no Bolt it reads, so a stale local Bolt lets a former owner push an intent; and the act rule waits only on pause record coverage, so a paused state reached by a replay still answers act
    fix: >-
      In D3, state that the owner read of the pause and cancel refusal is against a Bolt fetched by the verb, and what it does with origin unreachable (F7).
      In D1's act rule, add that the FSM states paused and cancelling answer wait, beside the pause record coverage, so an intent that reached the owner through a replay holds the run.
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:66
    summary: D2 retires only the per-transition push spawn behind statuscommit, but SpawnStatusPush also serves the push deferred to a teardown lock's release, the anchor change and tender's prime push, so a transition whose push waits behind a teardown lock still lets the run act first
    fix: >-
      In D2's second bullet, state the rule for every detached status push, not only the statuscommit one:
      a transition's push deferred behind a held teardown lock waits for it synchronously before the next producer starts,
      and name whether the anchor change's and tender's prime push stay detached and why neither carries a run transition.
  - id: F10
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:265
    summary: The old owner's orch keeps mirroring until its next Bolt pull, so its push lands after the adopter's restore; the adopter's later mirrors lease on the value read at restore, lose the lease and write nothing, and a later restore gets the old owner's transcript
    fix: >-
      In D10's mirror bullet, state that a mirror whose lease is lost re-reads the ref and, when it names another machine,
      writes nothing until this machine's next orch start restores or keeps per the restore bullet;
      and that the act rule's hand_over reads the Bolt owner, so the old owner's mirror stops once its takeover is read.
      State in D10's Bound how long the old owner's mirror can land after an adopt: up to the dobby's Bolt pull interval.
  - id: F11
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:331
    summary: The Open risk on D10 defers the weft repository's visibility, a fact the writer could read now, to the parent together with the operator's choice
    fix: >-
      Read the weft repository's visibility, for example with gh repo view and its visibility field, and record it under D10's Facts with the date read.
      Narrow the Open risk to the choice the parent makes: whether the operator allows the orch's transcripts in a repository of that visibility.
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:193
    summary: D7's Bound says a machine that has not read the hold runs only its step in flight to its transition, but its act rule still answers act, so it starts producers and pushes transitions until its next Bolt pull
    fix: >-
      In D7's Bound, replace "until then its step in flight runs to its transition" with
      "until then it goes on with its runs as before, for at most the dobby's Bolt pull interval; D4's bound covers the transitions it pushes".
  - id: F13
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:208
    summary: D8 step 1's rejected-push re-read refuses only an owner or hold of these subjects, but holds are hub-wide and carry no subjects, so two machines adopting different beams at once can both push a hold
    fix: >-
      In D8 step 1, change the re-read to refuse over an owner of these subjects or any hold another machine wrote since the read, naming that machine.
  - id: F14
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:188
    summary: Today's adopt pause also covers unstarted released tasks, which have no batten record, so D7's conversion into a run pause intent kept in the batten record has nowhere to write it and a written record would take the task out of queued
    fix: >-
      In D7's upgrade bullet, state what a standing adopt or adopt-hold entry of a released task with no batten record becomes:
      either it is dropped, the task staying queued, or the conversion writes a board-side hold the act rule reads; pick one and say it.
  - id: F15
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:248
    summary: An orch Claude process launched by the pre-upgrade lym is not in runproc, so the first orch start after the upgrade refuses over it naming its pid, the A6 and A7 case again, and no bound states it
    fix: >-
      Add to D9 what the first orch start after the upgrade does with a live holder lym launched before the registration existed,
      such as reading it as this worktree's orch when its command line and cwd match the worktree's orch launch,
      or state in D9's text that the refusal stands then and the operator exits that session once.
  - id: F16
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:220
    summary: The try count that bounds adopt_fix_tries lives in the adopt subject directory under Bolt's .lym, which D3 classes as cache that reads as empty, so a lost or rebuilt cache resets the limit
    fix: >-
      In D8 step 8, state where the try count lives: in the adopt-fix hold in Bolt, written with each filing,
      so a re-check on any machine reads the same count; or state in D8's Bound that the count is per machine and resets with the cache.
  - id: F17
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:337
    summary: The last Open risk on claude-account-pick-simple restates the Scope rule that D10 restores into whatever config directory the claude engine resolves, and asks the parent nothing
    fix: >-
      Delete the last Open risks bullet, the one naming claude-account-pick-simple; the Scope Out bullet already carries the rule.
  - id: F18
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:327
    summary: The second Auto-mode assumption points to the question ledger, which the plan writer never reads, and adds nothing the Decisions do not state
    fix: >-
      Delete the bullet "The other picks the question ledger records stand as D1 to D11 state them." from Auto-mode assumptions.
---

### [BLOCKING] D4's hand-over reads a replay that conflicts

**Location:** `_lym/discussion/decision-record.md:116`, D4 bullets 2 and 4.

**Issue:** The takeover entry goes into the batten record and the child record, and every transition of the old owner edits those same records.
`fabricengine`'s records replay stops on a conflicting pick with `ErrReplayConflict` and restores the starting tip (`internal/fabricengine/replay.go:45-48`).
So in the common case there is no replay result to read the takeover from:
the old owner never hands over, keeps its processes and archives nothing, and D4's first acceptance criterion cannot pass.
The adopter's own rejected takeover push hits the same conflict.
D4's "replays as today" also contradicts the declared `PATTERN-run-fsm` text, "hands over, never replays".
All four lenses reported this.

### [BLOCKING] The hold's hub pause entry holds what the hold lets act

**Location:** `_lym/discussion/decision-record.md:180`, D7 bullets 2 and 3, D8 steps 5, 7 and 8.

**Issue:** D7 has every machine that reads a hold, the adopter included, write a hub entry in its pause record.
A hub entry covers every slug (`internal/pauserecord/doc.go`).
So the adopt-fix task's agent turns and network calls wait at the gate, although the act rule answers `act` for that task.
The deploy and check children wait too, since `SetGateExemptAll` never reaches a child.
The orch watcher holds the cycle machine and interrupts a busy orch under any live hub entry (`internal/orchengine/pause.go`), against D8's orch kept up during the hold.
`lym batten cancel` refuses while a hub pause covers the prime (`internal/battencli/cancelcmd.go`), which blocks the fix loop's own way forward for a halted fix task.
The fix loop cannot land, so the hold never clears.

### [BLOCKING] A hold no live adopt owns has no way out

**Location:** `_lym/discussion/decision-record.md:195`, D7 Bound and D8 step 1 and its `--release` paragraph.

**Issue:** Take an adopter that is switched off after step 1, or one whose `adopt-fix` hold stands when it goes off.
Every other machine then reads the hold and waits for every run.
`lym adopt` on another machine is refused over the hold, naming the dead machine.
`--release` is defined only for "a hold over a failing check", and no Decision lets a machine clear a hold another machine wrote.
That contradicts the Goal's "go on later … on another one after it pulls".
The record states no bound for a hub-wide guard whose writer is gone.

### [MEDIUM] A landing past its parent-side merge is stranded under a hold

**Location:** `_lym/discussion/decision-record.md:51`, D1 act rule and D7 bullet 2.

**Issue:** The carve-out applies to `hand_over` alone.
D7 makes the act rule answer `wait` for every run on every machine, and a hub entry is the one pause entry that holds a run holding a parent lock.
So a Finalize past its merge freezes with its parent lock taken for as long as the hold stands.
At the fix-try limit the hold stays, so the landing stays frozen indefinitely.
D6 and D9 say such a landing finishes here, and `PATTERN-owner-machine` says a landing in flight is never stranded.

### [MEDIUM] The hold's local entry shares the hub slot with the operator's and hibernate's pauses

**Location:** `_lym/discussion/decision-record.md:182`, D7 bullet 3.

**Issue:** The pause record holds at most one live entry per scope.
A hold's hub entry therefore collides with the operator's hub-wide `lym tender pause` and with hibernate's hub pause.
D7's removal of that entry can lift the operator's own pause.
`lym tender resume` lifts the hold's entry while the Bolt hold stands (`internal/tenderengine/resume.go`).
The hibernate wake skips its origin reconcile under any foreign hub entry, and its notice tells the operator to lift the entry (`internal/hibernateengine/wake.go:58-64`).

### [MEDIUM] Cancel removes the record D3 puts its intent in, and records no finish

**Location:** `_lym/discussion/decision-record.md:88`, D3 bullet 1, D1 states 2 and 3, D11 bullet 2 and its Bound.

**Issue:** `lym batten cancel` removes the run's records and commits that removal (`internal/battencli/cancelcmd.go`).
D3 keeps the cancel intent in that batten record without saying the record survives, and it names no mark for a finished cancel.
D1 cannot then tell `cancelled` from `cancelling` from tracked records.
D11's kept pair also rests on a batten record the cancel deletes, so after its entry is removed it reads as an orphan and is torn down against the keep.

### [MEDIUM] No Decision says what a slug pause or a cancel does offline

**Location:** `_lym/discussion/decision-record.md:90`, D3 bullets 1 and 2, D2 Bound.

**Issue:** D3 makes `lym batten cancel` push its intent before it ends anything, and `lym tender pause <slug>` and `resume` push before they return.
Today an offline cancel ends the run's processes and leaves a marker.
Under D3 an offline cancel can end nothing, and the record does not say whether these verbs refuse, block or act locally.
Only the machine-local hub pause in the Scope stops an in-flight step offline, and no Decision names it as the offline stop.

### [MEDIUM] The owner-only intent guard reads an unnamed Bolt, and the act rule misses a replayed pause

**Location:** `_lym/discussion/decision-record.md:97`, D3, and the act rule at `decision-record.md:49`.

**Issue:** A former owner whose local Bolt still names it passes the refusal and pushes a pause intent.
The new owner picks the intent up through a replay.
The act rule's pause input is this machine's pause record coverage, which only the verb and a catch-up write.
So D1 reads `paused` while the act rule answers `act`.
The guard's bound is unstated.

### [MEDIUM] Other detached status pushes survive D2

**Location:** `_lym/discussion/decision-record.md:66`, D2 bullet 2.

**Issue:** D2 retires only the per-transition spawn behind `statuscommit`.
`SpawnStatusPush` also serves the push deferred to a teardown lock's release (`internal/fabricengine/teardownlock.go:169`),
the anchor change (`anchorchange.go:557`) and tender's prime push (`internal/tendercli/deps.go:148`).
A transition whose push waits behind a teardown lock still lets the next producer start before origin has the records,
against "a transition pushes its records before the run acts".

### [MEDIUM] The old owner's mirror outruns the adopter's restore and starves its lease

**Location:** `_lym/discussion/decision-record.md:265`, D10 mirror and restore bullets.

**Issue:** The old owner reads the takeover only at its next Bolt pull, so until then its orch mirrors at turn end, under a lease on its own last value, and lands after the adopter's restore.
The adopter's mirrors lease on the value it read at restore, lose the lease and write nothing, and nothing says they re-read.
A later restore then gets the old owner's transcript.

### [MEDIUM] D10's Open risk defers a fact the writer could read

**Location:** `_lym/discussion/decision-record.md:331`, Open risks bullet 1.

**Issue:** The weft repository's visibility is a fact, readable now from GitHub, and the design's risk rests on it.
The rubric's deferred-research rule asks for it to be researched and recorded with its evidence.
Whether the operator allows transcripts there is a choice and stays with the parent, so only the fact half is the defect.

### [LOW] D7's Bound understates what a machine runs before it reads a hold

**Location:** `_lym/discussion/decision-record.md:193`.

**Issue:** Until its next Bolt pull, which the state map puts at up to 5 minutes (`dobbyengine/handover.go`), a machine's act rule answers `act`.
It goes on starting producers and pushing transitions, not only its step in flight.
D4's bound covers the outcome, so only D7's wording is wrong.

### [LOW] D8 step 1's re-read refuses only holds of these subjects

**Location:** `_lym/discussion/decision-record.md:208`.

**Issue:** D7 makes holds hub-wide with no subject.
So the rejected-push re-read never matches a hold another machine wrote for a different beam, and two adopts can both land a hold.
That contradicts the up-front refusal in the same step.

### [LOW] The upgrade conversion has no record for a released task

**Location:** `_lym/discussion/decision-record.md:188`.

**Issue:** `PATTERN-dobby-runs` has today's adopt pause cover unstarted released tasks too, and those have no batten record.
D7 turns each such entry into a run pause intent that D3 keeps in the batten record.
Writing one would take the task out of `queued`, the one state D1 gives a task without a batten record.

### [LOW] The first orch start after the upgrade refuses its own pre-upgrade orch

**Location:** `_lym/discussion/decision-record.md:248`.

**Issue:** An orch launched before D9's runproc registration exists is not registered.
So D9 refuses over it naming its pid, which is the A6 and A7 case the decision exists to end, and nothing states that bound.

### [LOW] The fix-try limit lives in a cache

**Location:** `_lym/discussion/decision-record.md:220`.

**Issue:** The try count is read from the adopt subject directory under Bolt's `.lym` (`internal/adoptcli/fixloop.go`, `readTries`).
D3 lists adopt check views among the caches that read as empty, so a lost cache, or a re-check on another machine, resets the limit D8 relies on.

### [LOW] An Open risk restates the Scope

**Location:** `_lym/discussion/decision-record.md:337`.

**Issue:** The Scope already settles the rule that D10 restores into whatever config directory the claude engine resolves for the worktree.
The last Open risks bullet repeats it as a "must" and asks the parent nothing, so it is no open risk.

### [NIT] An Auto-mode assumption points at the ledger

**Location:** `_lym/discussion/decision-record.md:327`.

**Issue:** The plan writer reads nothing of the support log, so a pointer to the question ledger carries nothing it can use.
The picks it names are already the Decisions.

## Rejected

- bound-breaker F9, two machines with one current name register as one: a name collision predates this design, and no identity beyond the name exists for D5 to tell them apart; D5 claims only to join a machine's former names.
- D8 step 8's filing through `fronttask.File` (focus item 5): not a defect. It is the existing adopt fix loop, which `lym adopt` runs in the operator's terminal and the dobby's on-land action reruns; no run's agent creates or promotes a task. No Decision edits this task's own board entry.
- Relocation (focus item 8): the Rationale paragraphs carry each Decision's reason and the A-numbers they settle. D10's Facts are the evidence the spec asks for. The only line to cut is F18.
- interactions finding 9, D4 against the declared `PATTERN-run-fsm` text: merged into F1, whose fix makes the two agree.
