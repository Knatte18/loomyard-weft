---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:86
    summary: >-
      A pass started while a pause covers the run is refused by batten's pre-run (`refuseCovered`) before InnerRun runs,
      so D4's yield under a pause, the owner re-check in the spawn hold under an owner pause, and two acceptance criteria describe a path no pass reaches,
      and every covered run's probe-tick pass becomes a failing pass.
    fix: >-
      In D4, add one decision: dobby runs no pass of a run while a pause entry covers the hub or the run's slug (it reads the same pause record it already reads for D5 and D12),
      and a `lyx batten run` pass refused at pre-run for a covering pause (`battenPreRun` in `internal/battencli/arm.go`) is read as covered, never as a failing pass, and counts toward neither the three-failures notice nor D7's failing-pass marker.
      The pass at the lift stays as D4 and D12 say.
      Then correct what rests on the false premise.
      In the spawn-hold paragraph, say the hold yields only for a pause that begins during the pass or a standing owner no pause covers (an owner pause that failed to establish), and drop "dobby goes on passing it" from the sentence on an owner InnerRun finds inside a continue pass, replacing it with: dobby passes it again at the lift of the owner pause, like any covering pause.
      Rewrite the acceptance bullet "A batten pass at Run-Shed under a covering pause or a standing owner yields at once" to the cases that are reachable (a pause that begins mid-pass; a standing owner with no covering entry), and add a bullet that a covered run gets no pass and no failing-pass notice.
      If the record instead wants passes to start under a pause, it must say so and name the change to `battenPreRun` and to the hand-run refusal that names `lyx resume`.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:102
    summary: >-
      The owed full pass of an origin-unreachable resume is recorded only by dobby reading its own pass's `origin unreachable` output line,
      so a hand pass, `lyx beam land`'s detached first pass or any pass dobby did not spawn that resumes on the local Bolt owes nothing, and a takeover during the outage is never judged against origin.
    fix: >-
      In D4, make the pass itself record the episode: a full pass that resumes with origin unreachable writes a per-run owed marker (a tolerant record under the run's scratch directory, named through `internal/lyxdirs`) before it goes on,
      and a full pass that reaches origin and exits 0 removes it.
      Dobby reads the marker (a full pass is owed while it exists, with `--no-reopen`) instead of parsing an output line, which drops the output-line protocol from the record.
      Reword the acceptance bullet "A full pass owed after a start or a wake stays owed until a full pass of the run that reached origin exits 0" to cover a hand pass and a detached first pass.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:108
    summary: >-
      A run marked owned elsewhere whose owner pause failed has no pass to re-judge it and no lift to clear the mark,
      so dobby retries the pause on each probe tick without checking the owner and can pause and end the processes of a run that `lyx adopt` has since taken back here.
    fix: >-
      In D4, replace the blind retry of a failed owner pause with a verdict: while the mark stands and no pause entry covers the run's slug, dobby runs a full pass of the run on the probe tick instead of retrying the pause.
      That pass either is refused as owned elsewhere again, in which case dobby runs the owned-elsewhere handling again (pause, then the marker keeps the notice to once per episode), or exits 0, which clears the mark.
      The retry then never pauses a run on a stale verdict.
      Add to the owned-elsewhere acceptance bullet that a run adopted back while its owner pause had failed is not paused.
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:61
    summary: >-
      D2's refusal covers an adopted member task whose parent beam is owned elsewhere, but `lyx adopt` of a beam (or any pair) whose own parent beam or prime reads owned elsewhere leaves the beam's landing run, whose parent is that beam, with no dobby that both owns it and is its parent's.
    fix: >-
      In D2, widen the first refusal from a member task to any entry the call records an owner for, task or beam, whose parent reads owned by another machine: it refuses naming `lyx adopt <parent>` (or `lyx adopt --prime`) unless the same call adopts the parent.
      `adoptcli`'s `parentWarning` already judges every entry this way, so this turns that warning into the refusal for the beam case too.
      State that the test is made against the entries the call actually records, so a parent the same call stops before its owner write (a diverged or live pair at step 1) does not count as adopted.
      Add the beam case to the acceptance bullet on `lyx adopt`'s refusals, and to Q21 in the support log.
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:175
    summary: >-
      D5 names `lyx board set-status <slug>` with no status as the release, but the verb takes a JSON payload whose `status` key is required and refuses an absent key (only an explicit null clears), so the release command D5 and the start guard's refusal name does not work as written.
    fix: >-
      In D5, name the release as `lyx board set-status '{"slug":"<slug>","status":null}'` wherever it appears (the release sentence, the start-guard refusal that names it, and the acceptance bullet on a hand-run start of a draft),
      or decide to change `set-status` to accept a missing status, say so, and note it in the board module's docs.
      The refusal message must name a command that works as typed, per `PATTERN-refusal-way-forward`.
      Correct Q6 in the support log, which calls this an existing verb used with no status.
  - id: F6
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:172
    summary: >-
      D5 enumerates the creating writes that give `draft` and claims no other path leaves a task statusless, but an upsert that changes an existing project to a task (the store has no kind-change guard) and a write that sets an empty-string status both leave a task with an empty status that dobby would start unreleased, and D5 never says whether empty means nil, an empty string or both.
    fix: >-
      In D5, replace the list of creating writes with one rule applied where the store commits an entry: a write that leaves an entry of kind task with an empty status gives it `draft`, unless the write is a release, meaning a payload that sets `status` to null or a `set_status` step with a null status on an entry that was already a task.
      State once that empty means a nil status or an empty string, and apply it in the start condition, the migration and the release paths alike.
      Name the project-to-task upsert and an empty-string status among the acceptance cases, and drop the sentence "No code path but an explicit release leaves a task statusless" or restate it as the rule above.
---

### [BLOCKING] A pass under a covering pause is refused before InnerRun, not yielded

**Location:** `_lyx/discussion/decision-record.md:86`, `:111`, `:306-307`, `:258-260`; code at `internal/battencli/arm.go:644-647`.
**Issue:** `battenPreRun`, the PreRun hook of `lyx batten run`, calls `refuseCovered` first and returns the "lyx resume" refusal when a live pause entry covers the run's slug.
D4 schedules passes on a child status write, on a batten status write and on every probe tick, and says nothing of coverage for a running run; D5 and D12 apply coverage only to a start and to the lift.
So under any hub or slug pause, including the owner pause InnerRun itself makes, each of those triggers runs a pass that exits non-zero before InnerRun, and the run is retried on each tick as a failing pass.
D7 then posts a failing-pass notice after three equal errors per covered run, which contradicts the once-per-episode owner notice and puts a notice per run on every hub pause.
The rest of D4's pause design assumes InnerRun runs under the pause: the spawn hold's yield with the reason printed, the owner re-check inside that hold, "dobby goes on passing it" for an owner InnerRun finds, and the acceptance bullets "A batten pass at Run-Shed under a covering pause or a standing owner yields at once, printing the hold reason" and "A run held at the spawn on a standing owner and then adopted back is respawned by the next pass with a due probe".
Under an owner pause the second can only be reached when the owner pause failed to establish, so the re-check that focus key 2 asks about is unreachable in the case the paragraph is written for; the lift pass is what takes the run back, and there the spawn hold waits for the next due probe, which is the stated one-probe latency bound.

### [MEDIUM] The owed full pass of an unreachable-origin resume is invisible to a hand pass

**Location:** `_lyx/discussion/decision-record.md:102`, `:308`.
**Issue:** D4 has the full pass print an `origin unreachable` line that dobby reads, and dobby holds the owed state in its own record.
A resume of a halted run is by hand pass, the very case D4 sends through the origin verdict, and `lyx beam land` and `lyx adopt` start detached first passes.
None of them is spawned by dobby, so none leaves an owed episode in dobby's record when it resumes against the local Bolt.
Dobby's next pass reads a record with no full pass owed and runs continue passes, so a takeover during the outage is not judged against origin until the next dobby start or wake, which is what the owed-pass rule was written to prevent.
The line protocol is also a parsed output text, where the other pass outcomes use exit codes.

### [MEDIUM] A failed owner pause is retried on a stale verdict

**Location:** `_lyx/discussion/decision-record.md:108-110`.
**Issue:** While the mark stands dobby runs no pass of the run, and the mark clears only at the full pass at the lift of a covering pause.
When `PauseOwnedElsewhere` failed there is no pause entry and so no lift, and `lyx adopt` then has nothing to lift; D4 still has dobby retry the pause on each probe tick.
Today's InnerRun re-checks the owner before each retry (`foldOwnerTrigger`'s retry trigger runs `ownerStep`); dobby's retry does not.
After an adopt the retry pauses a run this machine owns, and `endOwnedElsewhere` ends its registered processes, which only a manual `lyx resume` or a second adopt undoes.

### [MEDIUM] D2's adopt refusal omits a beam whose parent is owned elsewhere

**Location:** `_lyx/discussion/decision-record.md:61`, `:299`; support log Q21.
**Issue:** D3 gives a beam's landing run to its parent's dobby, which drives only runs its machine owns.
A beam adopted onto this machine while its parent beam or the prime is owned by another machine has an owner on one machine and a parent's dobby on the other, so neither drives its landing run; this is the shape round 2's F7 closed for member tasks.
`adoptcli`'s existing `parentWarning` already treats any entry with a parent owned elsewhere alike, and D2 narrows its refusal to "a member task".

### [MEDIUM] The draft release names a command that fails as written

**Location:** `_lyx/discussion/decision-record.md:175`, `:177`, `:301`; support log Q6.
**Issue:** `lyx board set-status` takes a JSON payload and requires the `status` key; `internal/boardcli/cli.go` says an absent key is refused so a typo cannot clear the status, and only an explicit null clears it.
D5's "`lyx board set-status <slug>` with no status clears it" and the start guard's refusal naming `lyx board set-status <slug>` therefore give an operator a command that errors with "json payload required".

### [MEDIUM] D5's list of draft-giving writes is not the closed set it claims

**Location:** `_lyx/discussion/decision-record.md:172-176`, `:189`.
**Issue:** Checked against `internal/boardengine/store.go` and `task.go`: `UpsertTask` accepts `kind` on an existing entry with no kind-change guard, so a project upserted to `kind: task` becomes a task with no status, a path D5 does not list (its list stops at a note turned into a task).
`status` also accepts an empty string, which `Task.Status` stores as a non-nil pointer that is written to `board.json`; whether that counts as an empty status for the start condition, the migration or a release is not stated.
The closing claim "No code path but an explicit release leaves a task statusless" is false as written, and it is the premise of D5's bound.
A single rule at the store's commit point removes the need to enumerate writes, which also closes the enumeration gap as a method rather than item by item.

## Rejected

- Focus key 1, owner and retry once: the three owned-elsewhere arms D4 names (`resumeRefusal`, `resumeOwnerClaim`, `refuseResumeUnreachable`) are the resume refusals in `internal/battencli`, and giving them their own exit code, a persisted mark, one pause, one lock release and one notice per episode means the run is not retried as a failing pass; the open point is F3 and the pause interaction in F1.
- Focus key 3, continue pass: D4's continue-pass paragraph, the `--continue-pass` bound and the PATTERN changes section agree with the declared `PATTERN-owner-machine` text, and no path found lets a resume skip the verdict (a retried pass into a run not at Run-Shed is a full pass under D4's bound, and dobby observes a sleep on each tick before it schedules a pass).
- Focus key 5, one check per pass: D4 names `check()`'s full body and the list matches `internal/battenshed/innerrun.go` (`coverage`, `foldOwnerTrigger`, `checkChild`, `ownerStep`, `refreshWatched` and `lockStep` on a due probe, `noticeStep`, the arm step, the pause-flag read), and the inventory's last bullet and the counters bullet account for `ownerSleep`, the `childWait` sleep watch, the respawn count and the api-error marks and counts.
- The writer/reviewer symmetry note, the rejected-alternatives rule and the missing "Notes for the plan writer" are not flagged; no Decision edits this task's own board entry or files or promotes a task.
