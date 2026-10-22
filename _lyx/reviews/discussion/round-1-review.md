---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D7 bullet 3
    summary: D7's halt names ways forward that do not lead to a landing - adopt --prime is wrong for a pair parent, nothing resumes the halted run after adopt --prime, and a failed run-owner check gets a parent-only message (PATTERN-refusal-way-forward)
    fix: Rewrite D7 bullet 3 so the message depends on which check failed. When this machine does not own the parent, the message names the parent's owner and two ways forward. The first is adopting the parent here, `lyx adopt --prime` for the prime or `lyx adopt <parent-slug>` for a pair parent, followed by the resume that lands the halted run (name the verb, for example `lyx loom resume` from the pair). The second is `lyx adopt <slug>` on the parent's owner. When this machine does not own the run, the message names the run's owner and `lyx adopt <slug>` here, then the same resume. Also state whether D5 step 5 resumes a child halted at the owner halt whose parent `lyx adopt` just adopted here. If it does, D5 step 5 must say so for `--prime` and for a parent slug. If it does not, the message must carry the resume.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D4 bullet 3, D5 steps 1, 3 and 5
    summary: D4's fast-forward-only guard lapses when adopt flips the owner, so an old owner adopting back replays the local commits it made after the earlier handover, and a diverged code branch left by step 1 makes step 5's resume refuse with no stated way forward
    fix: In D5, add a rule between step 1 and step 3. A named slug whose local records or code branch step 1 judged needs_replay, ahead, diverged, rewound or rewritten stops before its owner write. Its envelope entry names the branch, its state and the way forward (for example removing the stale local pair through fabric and re-adding it from origin), and the other slugs go on. Add an acceptance criterion that adopting back a slug with local commits made after an earlier handover records no owner and replays nothing.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D7 bullet 2
    summary: The parent lock that a run halted at Finalize keeps is never released once the parent's owner adopts that run, because batten's process stays alive under the owner pause, so a later adopt of the parent here leaves every other run queued behind it
    fix: In D7 bullet 2, state the release. When batten's owner check (D4 wake, D6) finds its run owned elsewhere, it releases the parent lock its run holds, through `parentlock` Release by its own owner process, before it adds the owner pause. Add that the released run takes the lock again at its next lock-taking row if it is ever adopted back. Add an acceptance criterion that a run halted at Finalize and adopted by the parent's owner leaves no parent lock held on the halting machine after that machine's next owner check.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D6 bullet 2, D4 bullet 4 (wake and orch start pauses)
    summary: A slug-scope owner pause does not hold a run that holds the parent lock, because pauserecord exempts such a caller from slug entries, so a run between Publish and PR-Gate on the old machine keeps pushing and approving after the handover, and no bound states it
    fix: In D6 bullet 2 (and D4's wake and orch-start pause bullets), state what holds a run that holds the parent lock when it reads owned elsewhere. Either the owner check first releases the lock (as F3's fix adds) so the slug entry then holds the run, or a separate halt of the row is named. Put the remaining window in D6's Bound line and in Open risks.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D3 bullets 2 and 5
    summary: D3's owner refusal on lyx loom start for an existing run also reaches batten's own InnerRun bootstrap, revive and re-step spawns of lyx loom start --no-attach, which handle only two named spawn errors, and D3 does not say how the shed verbs read an unreachable origin
    fix: "In D3, add a bullet for batten's own child spawns. Either an owner refusal from a child spawn InnerRun makes is read as an owned_elsewhere verdict, taking D6's owner pause and notice, or InnerRun's spawns skip the shed verbs' owner check because batten already judged the run. State which. Extend D3 bullet 5 to the shed resume verbs: with origin unreachable, a task this machine owns on the local Bolt resumes with a Warn, and every other case is refused naming the re-run."
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D3 bullets 1, 3 and 5
    summary: Batten's board claim happens inside the Worktree-Create row after the run is seeded and only warns on failure, so a start check placed after the claim's pull cannot refuse before create and seed, a refused start leaves a running status on another machine's entry, and refusing an unreachable start contradicts the warn-only claim without saying so
    fix: In D3 bullet 1, place the start's owner check before seeding. Batten pulls Bolt and reads the effective owner before it writes a seed, and the owner is then written by the claim. State that a start's claim, which today only warns on a board failure, now refuses the row when the owner write or its push fails, and that the stopped run resumes with the same verb.
  - id: F7
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D5 Arguments bullet and step 5
    summary: The --all flag names every task that has an owner, done but unpruned tasks included, while a done slug refuses the whole verb before anything runs, and step 5 starts a fresh run of a named task with no run record, beyond the board's resumes the adopted runs
    fix: In D5's Arguments bullet, define `--all` as the prime and every task that is not done and has an owner or a run status. In step 5, start or resume only a task that has a batten run record that is not done. A named task with no run record gets its owner recorded and no run, and the envelope says so.
  - id: F8
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md Constraints, PATTERN-origin-reconcile line
    summary: "The Constraints line says the only owner reads outside ReconcileOrigin are a start's read and Finalize's parent read, but D3's unreachable-origin resume, D5's post-push re-read and --all selection, and D9's status reads also read owners, and D3 leaves unstated an unreachable origin with another owner on the local Bolt"
    fix: Reword the Constraints line to state the rule as a class. The owner verdict that gates a resume, a wake or a landing against origin is computed inside `ReconcileOrigin`. Every other owner read reads the local Bolt, namely a start's read, Finalize's parent read, adopt's selection and post-push re-read, a resume with origin unreachable, and the status views. In D3 bullet 5, add that with origin unreachable a task the local Bolt shows as owned elsewhere is refused, naming the owner and a re-run once origin answers.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D3 bullet 4, D2 bullets 1 and 6, Open risks bullet 3
    summary: D3's line about a resume of a task with no owner recorded reads as covering the shed resume verbs, while D2's setter callers and its run-held rule name batten's resume only, and Open risks says until its next resume, which understates the window if loom resume records nothing
    fix: In D3 bullet 4, write batten's resume, and add that the shed resume verbs on a run-held task with no owner recorded proceed without writing an owner. In Open risks bullet 3, write until its next batten resume.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D4 bullet 4, D5 step 4
    summary: D4's marking conflates hibernate's Caller label with the entry identity hibernate stores in its request, and adopt has no stored identity, and a slug already paused gets no new marked entry because tender adds no second live entry per scope, so adopt lifts nothing for it
    fix: In D4 bullet 4, state that owner pauses carry one fixed `Caller` label, and that an owner check finding the slug already paused leaves the standing entry and adds none. In D5 step 4, state that adopt lifts each live entry carrying that label whose scope is an adopted slug, by that entry's scope, caller and paused time. A slug still covered by another entry gets an envelope line naming the standing pause and `lyx resume <slug>`.
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D3 bullet 4, D2 bullet 4
    summary: An owner-writing resume or a start's claim whose push succeeds while the replay kept origin's owner proceeds on a task another machine owns, since D3 re-reads nothing after its push, unlike D5 step 3
    fix: In D3 bullet 4, add that after the owner write's push the start or resume re-reads the owner from Bolt. An owner that does not read as this machine refuses, naming the owner and `lyx adopt <slug>`, as D5 step 3 does.
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md Acceptance criteria
    summary: No criterion covers D1's empty-hostname error, D3's unreachable-origin refusal of a start or owner-writing resume, D4's Summary naming each owned_elsewhere entry, D6's retry with origin unreachable, or D9's README Running rows and orch stencil reading
    fix: "Add criteria: an empty hostname is an error and writes or compares no owner; a start or an owner-writing resume with origin unreachable is refused naming the re-run, while a resume of an owned task warns and resumes; the reconcile Summary names each owned_elsewhere entry with its owner; InnerRun with origin unreachable after a sleep retries at a later check and holds nothing; the README's Running rows name the owner, and the orch stencil reads a run owned elsewhere as that machine's."
  - id: F13
    severity: LOW
    class: decision
    origin: handler
    location: _lyx/discussion/decision-record.md D8 bullet 1
    summary: D8 opens a window on every resume past Open-Window with no client attached, including same-machine resumes after the operator detached or an orch-driven resume, a behaviour change beyond issue 742 that D8 does not state as intended or bound
    fix: In D8, either state that a same-machine resume with no client attached also opens a window, as intended, and add a criterion for it, or restrict the reopen to a resume whose pair this machine did not drive last, and say how that is read without a marker.
  - id: F14
    severity: NIT
    class: consistency
    origin: handler
    location: _lyx/discussion/decision-record.md D7 bullet 3, second sentence
    summary: The clause about rewording the board's let the owner land the run onto built pieces, as beam work, is deliberation narrative the support log's Turn 4 already holds
    fix: In D7 bullet 3, delete the clause from This rewords up to beam work. Keep its behavioural remainder as its own sentence, The halting machine's run is then owned elsewhere and paused at its next owner check. The support log's Turn 4 goal-scope line already records the rewording.
---

# Round 1 review — machine-handover discussion

The record's bounds are thorough, but four mechanisms break when run end to end: D7's way forward, adopt flipping D4's guard, the parent lock a halted or locked run keeps across a handover, and the shed-verb check meeting batten's own child spawns.

### [BLOCKING] D7's halt names ways forward that do not land the run

**Location:** `_lyx/discussion/decision-record.md` D7 bullet 3.

**Issue:** The message always names `lyx adopt --prime` here.
D2 defines a pair parent whose owner is that pair's task's owner, and for such a parent `--prime` adopts the wrong entry.
After `lyx adopt --prime`, D5 step 5 starts or resumes only adopted tasks, and `--prime` adopts no task, so the run halted at Finalize stays halted.
A halted child waits in InnerRun for the orch (`internal/battenshed/doc.go:115`), and the message names no resume.
When the failed check is the run's own owner, both named ways forward concern the parent.
The record's own Constraints line and `PATTERN-refusal-way-forward` require a halt that names a working way forward.
All four lenses found this independently.

### [MEDIUM] Adopting back replays the old owner's post-handover commits

**Location:** D4 bullet 3, D5 steps 1, 3 and 5.

**Issue:** Case B → A → B: B made records commits after A's adoption and before B's next owner check.
At B's `lyx adopt S`, step 1 still reads S as owned_elsewhere, so the needs_replay branch stays.
Step 3 makes S owned here, and step 5's batten resume runs a moving run-scope reconcile (`internal/battencli/resumereconcile.go` `reconcileRun`), which replays those commits onto origin's tip (`internal/fabricengine/doc.go:62`).
That is the publication D4 exists to prevent.
A diverged code branch left by step 1 makes the same resume refuse as stopped (`resumeRefusal`), so adopt's "starts its run" fails with no stated way forward.

### [MEDIUM] The parent lock a halted run keeps outlives its handover

**Location:** D7 bullet 2.

**Issue:** A parent lock stays valid while its owner process lives (`internal/parentlock/doc.go`, Liveness).
A halted child keeps batten's InnerRun waiting, and a covered holder keeps its parent lock (`internal/battenshed/doc.go:28`).
After the parent's owner adopts the run, the halting machine's next owner check adds an owner pause and the lock stays held with no end.
If that machine later adopts the parent, its other runs queue behind a holder whose run lives elsewhere, and only a force release frees it.

### [MEDIUM] A slug owner pause does not hold a run that holds the parent lock

**Location:** D6 bullet 2, D4's wake and orch-start pauses.

**Issue:** `pauserecord` exempts a caller whose run holds a parent lock from slug entries, so only a hub entry holds it (`internal/pauserecord/doc.go:23`, `hold.go` `SlugEntryExempt`).
A run between Publish and PR-Gate on the old machine keeps its entry points and network calls after the owner pause.
D7 lets PR-Gate run on a non-owner on purpose, so nothing stops its push or approve.
D6's "holds as under any pause" is false for this run, and no Bound or Open risk names it.

### [MEDIUM] The shed-verb owner check meets batten's own child spawns

**Location:** D3 bullets 2 and 5.

**Issue:** InnerRun's bootstrap, revive and re-step spawn `lyx loom start --no-attach` (`internal/battencli/wire.go:1160`).
On the old machine after a handover, D3's new refusal reaches that spawn as an unhandled spawn error instead of D6's owner pause and notice.
D3 bullet 5 states the unreachable-origin rule for batten only.
If the shed verbs refused while origin is unreachable, every offline batten run would fail at Run-Shed.

### [MEDIUM] The start check cannot precede the seed where the claim sits

**Location:** D3 bullets 1, 3 and 5.

**Issue:** The claim is written inside the Worktree-Create row's `CreateWorktree`, after the seed, and a board failure only warns (`internal/battencli/wire.go:778-784`, `internal/battencli/cli.go:26-28`).
A check "after the board's pull of its claim" therefore runs after seeding and after the claim's `running · Worktree-Create` write, contradicting "refuses before it creates, seeds, spawns".
A refused start would leave a run status on another machine's entry.
"A start with origin unreachable is refused" overturns the warn-only claim without saying so.

### [MEDIUM] `--all` contradicts the done refusal, and step 5 starts unstarted tasks

**Location:** D5 Arguments bullet and step 5.

**Issue:** `--all` names every task that has an owner, which includes a done task the prune has not removed yet.
The next sentence refuses a done slug before anything runs, so any such entry refuses the whole verb.
Step 5 reads a missing run record as not done and starts a fresh run, while the board entry's Design item 2 says adopt resumes the adopted runs.

### [MEDIUM] The Constraints line understates the owner reads outside the reconcile

**Location:** Constraints, the `PATTERN-origin-reconcile` line.

**Issue:** An unreachable reconcile returns no entries (`originreconcile.go`, the Unreachable early return), so D3's "a task this machine already owns" must read the local Bolt.
D5 re-reads owners after the push and selects `--all` by owner.
D9's tender listing covers run-held entries with no local pair, which the reconcile never judges, since it reports entries only for pairs with a local run record.
D3 also leaves open the case of an unreachable origin with the local Bolt naming another owner.

### [LOW] Shed resume verbs and the unowned run-held task

**Location:** D3 bullet 4, D2, Open risks bullet 3.

**Issue:** D2 lists only batten's claim and resume as setter callers and says the next batten resume records the owner.
D3's "a resume" reads wider.
A `lyx loom resume` on such a task passes the check, records nothing and drives the run, so Open risk 3's "until its next resume" understates the window.

### [LOW] The owner-pause marking is two identities stated as one

**Location:** D4 bullet 4, D5 step 4.

**Issue:** Hibernate marks its entry with `PauseCaller` (`internal/hibernateengine/request.go:44`) and lifts the exact entry stored in its request (`verbs.go` `liftOwned`).
Adopt stores no entry, so it can only select by caller label and scope.
Tender adds no entry on a scope that already has a live one (`internal/tenderengine/pause.go:111-116`), so a standing pause carries no owner marking and adopt lifts nothing for that slug.

### [LOW] An owner write whose replay lost proceeds unchecked in D3

**Location:** D3 bullet 4, D2 bullet 4.

**Issue:** Under D2, a push whose replay kept origin's owner succeeds and reports the local write lost.
D3 refuses only on a refused push, so the start or resume goes on for a task another machine owns.
D5 step 3 re-reads after its push, and D3 does not.

### [LOW] Acceptance criteria miss several Decisions' observable behaviour

**Location:** Acceptance criteria.

**Issue:** D1's empty-hostname error, D3's unreachable-origin refusal and Warn, D4's Summary naming, D6's unreachable retry, and D9's README rows and orch stencil reading have no criterion.
Every existing criterion traces to a stated Decision.

### [LOW] D8 changes same-machine resumes without saying so

**Location:** D8 bullet 1.

**Issue:** Today a run resumed past Open-Window opens no window (`internal/battenshed/doc.go:76`).
D8 opens one whenever no client is attached, which includes the operator having detached on the same machine and an orch-driven resume.
Issue #742 asked for per-machine counting, and D8 neither states the wider change as intended nor gives a criterion for it.

### [NIT] Deliberation narrative in D7

**Location:** D7 bullet 3, second sentence.

**Issue:** "This rewords the board's 'let the owner land the run' onto built pieces, since … beam work" explains a choice already recorded in the support log's Turn 4.
Its second half, that the halting machine's run is then owned elsewhere and paused at its next owner check, is behaviour the plan writer needs and stays.

## Rejected

- Board-entry writes (focus item 5): not a defect. D2's `owner_machine` exemption changes board-module code and writes a field outside the run lock's scope comparison (`internal/boardengine/runlock.go`), D5 writes owners and run statuses, and D9's README rows are generated board output, so no Decision writes this task's own entry body or scope.
- D6 re-firing after a hibernate wake (existing-paths trace): held, since the wake's slug pause still covers the slug, so no second notice goes.
- D7's claim that Merge-In, Publish, PR-Align and PR-Gate merge into no parent: held; the parent-side merge is Finalize's.
- D4's precedence order, D1's normalisation and D2's first-act prime claim with its replay race: held, and Open risk 2 covers the race.
