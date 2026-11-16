---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D1 (bullets 3 and 5) and D2 (bullet 2)
    summary: D1 retires only the teardown-lock respawn; the absorbing push lock's pending-flag respawn still pushes a flagged code side alone, so an old owner's code tip can land after a takeover and break the records-first order and D2's push-no-code-side guarantee
    fix: In D1, add a bullet that the pending-flag respawn in internal/fabricengine/pendingflags.go (coalesceFlagged, respawnForeignFlags) no longer pushes a pair side alone. A flagged pair side is either left for the pair's next synchronous push or respawned through the records-first PushPairAnchored with D2's takeover read. Add an acceptance criterion that a code side flagged by a push whose lock wait gave up never reaches origin before its records side, and never after a takeover entry naming another machine.
  - id: F2
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/discussion/decision-record.md Open risks and D9 rationale
    summary: D9 departs from the operator's stated basis for the stop, built on the run FSM's pause and cancel intents; only the run's parent can accept that departure
    fix: Raise to the run's parent whether the operator accepts D9's hub stop without per-run pause or cancel intents, resting its pushed-state guarantee on D1's synchronous pushes? Record the answer as a decision for adopt-build-resplit, then drop or keep the Open risks line to match it.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D6 Bound and the PATTERN-shell-guard-table constraint
    summary: D6's bound says only the operator's own shell runs --take-hold, but a shellguard row is a guardrail over static text and not a barrier, so an agent's script or quoted command substitution still reaches the flag
    fix: Rewrite D6's bound sentence. It should say that the shellguard row denies the plain command form to agents but is a guardrail over static text that a written script passes. The bound on a bypass is that --take-hold skips no check, lets no run go on before a green check, and that the old adopter stops at its next hold re-read naming this machine. Drop the claim that only the operator's own shell runs it.
  - id: F4
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md D5 Bound (No refusal or notice an agent reads names --take-hold), D6 last paragraph, Constraints (PATTERN-refusal-way-forward, PATTERN-shell-guard-table)
    summary: A shellguard row carries a Steer shown to the denied agent and a Notice in every agent's system prompt, so the new --take-hold row puts the flag in front of every agent, against D5's rule that no refusal or notice an agent reads names it
    fix: In D6, state what the --take-hold row's Steer and Notice say. Either give the row a Steer and Notice that name only that an adopt takeover of a hold is the operator's, without spelling the flag, and say the row's Forms entry is the only place the flag appears. Or narrow D5's bound and the PATTERN-refusal-way-forward constraint to except the guard row's own deny text, and say why that exception is safe.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker, lens:walkthrough
    location: _lym/discussion/decision-record.md D6 steps 1 and 8, and D5's upgrade first read
    summary: A plain lym adopt re-run on the machine whose own adopt-fix hold stands rewrites it as an adopt hold, dropping the fix task's exemption and the try counts, and the upgrade's hold carries neither the local try count nor an open fix task, though D5's notice and D10 both point at this re-run
    fix: In D6 step 1, add that a hold this machine wrote is kept as it stands, kind, fix task and try counts included, and that a re-run goes on from step 2. In D5's upgrade bullet, say that an open adopt fix task found at the first read makes the hold adopt-fix naming it, with the try count read from adoptcli's local tries file. Add an acceptance criterion that a re-run under this machine's own adopt-fix hold keeps the fix task and its try count.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D2 bullet on the code-tip grace
    summary: The rule that a machine owing a tip pushes it only while origin's last transition names it is a read then a separate push with no lease, so a tip the owner gave up meanwhile can still land on origin's code branch
    fix: In D2, make the owing machine's code-tip push a leased push, or make the give-up commit something the code push checks. One way is to push the code tip with a lease on origin's code branch at the value read with the records, and to re-read the records tip after the push, resetting origin's code branch with a lease when the transition no longer names it. Add a criterion that a tip given up between the owing machine's read and its push leaves origin's code branch where the owner's give-up left it.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D3 registration rules and Bound
    summary: D3 refuses only a former name equal to another machine's current name, so a new machine registering a current name that is another machine's former name, or two machines with one current name, both read an old owner as their own
    fix: In D3, add two refusals, each naming its way forward. A registration whose current name is a former name of another registered machine is refused, naming that machine and lym machine name <new> as the way forward. A registration whose current name is already registered by a machine with other former names is refused too. State how the registry tells two machines apart at registration, for example a machine id kept in machine.json beside the name. Add criteria for both refusals.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D7 upgrade bullet (A pair kept by a cancel before the upgrade has no batten record)
    summary: D7's upgrade premise is false; a cancelled run already keeps its batten record while its pair stands, and a legacy cancelled record with no cancel intent derives neither cancelled nor kept, so such a pair is neither listed as kept nor torn down
    fix: Replace the premise in D7's upgrade bullet with today's state, citing internal/battencli/cleanup.go (a cancelled run keeps its record while its pair stands) and internal/runfsm/state.go (Cancelled needs a cancel intent with a finish). Add to the first dobby start's upgrade that a batten record in shed state cancelled with no cancel intent gains a finished cancel intent with keep. Add an acceptance criterion that such a pair is listed as kept after the first dobby start and is not torn down.
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D5 hold slug entries against D1's owed-push retry
    summary: A run with an owed push answers wait and so gets a hold slug entry on a non-adopting machine, and D1's retry waits at the gate, so a hold freezes owed transitions until the adopter has taken the run from origin's older state, and the record says neither that this is meant nor which wins
    fix: In D5, choose and state one rule. Either a run's owed push is exempt from its hold slug entry, so owed transitions reach origin before the adopter's takeover. Or the hold freezes owed pushes, D2's archive covers them, and D5's Bound says so. Add a criterion for the chosen rule.
  - id: F10
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D2 hand-over bullets, D7 teardown condition, D4 step 4
    summary: D2's in-loop hand-over leaves the local pair standing, D7 skips pairs another machine owns and D4 removes them only after an absence, so on a dobby that stays up a handed-over pair stands unreported, the case D7's rationale names
    fix: In D2's hand-over list, add that the run's local pair is removed through internal/pairteardown's local mode, as D4 step 4 does, once its tips are archived. Add a criterion that after an in-loop hand-over no local pair of the slug stands.
  - id: F11
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D6 Retired list and D5 last bullet
    summary: lym beam status renders its adopt section from the adopt record D6 retires, and the record says nothing of what the status view shows of a standing Bolt hold, its machine or this machine's re-run
    fix: In D6, add that lym beam status replaces its adopt section with the Bolt hold, from internal/beamcli's adopt-section rendering. It shows the hold's kind, machine, time and subjects, the fix task and try counts under adopt-fix, and, for this machine's own hold, the lym adopt <subjects> re-run. Add a criterion for it.
  - id: F12
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D10 launcher bullet and its acceptance criterion
    summary: A hub's launcher is written only when this machine's prime dobby starts, so a machine that owns only beams under a prime another machine owns never gets a launcher, though D10's verb would start those beams
    fix: In D10, write a hub's launcher when any dobby of the hub starts on this machine for a worktree it owns, and write the all-hubs launcher with the first such hub. Change the matching acceptance criterion from the prime dobby's first start to the first owned dobby's start, and add a case of a machine owning only a beam.
  - id: F13
    severity: LOW
    class: decision
    origin: handler
    location: _lym/discussion/decision-record.md D8 closing paragraph
    summary: The entry's prime-is-root-beam member says every reader of an empty parent_beam, naming the orch role, the dobby and the board verbs, moves to the named root; D8 keeps the dobby's and board verbs' reads on the empty slug, a departure from the operator's text raised to nobody
    fix: Raise to the run's parent whether the operator accepts that only the printed parent_beam and its prose readers move to root, while the dobby's and board verbs' code reads keep the empty slug for the prime? Add the question to Open risks until the answer is recorded.
  - id: F14
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D2 second bullet (a takeover entry ... that this machine has not yet read)
    summary: The hand-over trigger depends on a per-machine record of takeover entries already read, which no decision names, places or gives a writer, so whether a repeated read hands over again is left to Plan
    fix: In D2, name where this machine records a takeover entry it has handed over on, for example a machine-local mark under .lym named in internal/lymdirs, written when the hand-over finishes. State that a later rejection reading the same entry skips the hand-over and pushes nothing for the run.
  - id: F15
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D4 step 4 against D2's hand-over and D2 Bound
    summary: D4 step 4 archives local tips as D2 does but omits D2's archive-tag push and its notice naming a dropped pause or cancel intent, so an intent owed offline on a machine that slept through a takeover is dropped silently
    fix: In D4 step 4, add that it pushes the archive tags and posts D2's one notice naming the machine and any pause or cancel intent it dropped.
  - id: F16
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D5 upgrade notice against D6 step 8
    summary: The upgrade turns adopt entries into run pause intents, yet its notice names only a re-run of lym adopt, after whose green check the runs still wait as paused
    fix: In D5's upgrade bullet, make the notice also name lym tender resume <slug> for each run whose entry became a pause intent, as the way to let it go on after the adopt's green check.
  - id: F17
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D1 teardown-lock bullet and PushOwedOffline
    summary: A push owed because a teardown lock stayed held is not offline, yet D1 reads it into PushOwedOffline, whose refusal says origin is unreachable
    fix: In D1, say which runfsm input a lock-owed push sets. Either a renamed input, PushOwed, whose refusal names the owed push and its last error, or PushOwedOffline with its refusal reworded to cover a held lock.
  - id: F18
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D2 Finalize-cut bullet and D5 landing-under-hold line
    summary: A Finalize cut between its local merge and its parent push has the merge only locally, so PastParentMerge reads false and the act rule waits under a hold, leaving the local parent branch ahead of origin for the whole hold
    fix: In D5, add that a run whose local parent branch holds its parent-side merge ahead of its upstream counts as past its parent-side merge for the act rule, so its parent push goes under a hold. Or state in D5's Bound that such a run waits out the hold with its merge local.
  - id: F19
    severity: LOW
    class: consistency
    origin: lens:existing-paths, handler
    location: _lym/discussion/decision-record.md D5 bullets on the hold caller and the legacy callers
    summary: The new pauserecord caller is named hold while pauserecord.HoldCaller already names the legacy adopt-hold caller that D5 keeps declared, so the record's term and the code's identifier point at different callers
    fix: In D5, name the new caller's Go constant and string distinct from HoldCaller, for example BoltHoldCaller with the string bolt-hold, or rename the legacy constant to LegacyAdoptHoldCaller in the same change. Say which in the bullet that introduces the caller.
---

### [BLOCKING] The pending-flag respawn still pushes a code side alone

**Location:** `_lym/discussion/decision-record.md` D1 bullets 3 and 5, D2 bullet 2.

**Issue:** `coalesceFlagged` marks the owed sides pending and, after its last run, respawns each side another pusher flagged as a detached push of that side alone (`internal/fabricengine/pendingflags.go:121-155`, `respawnForeignFlags` at :194).
D1 retires the per-transition spawn and the teardown-lock respawn but says nothing of this path.
A synchronous pair push whose lock wait gives up leaves its code flag set, and the holder's release pushes that code side by itself.
After a takeover this lands the old owner's code tip, breaking D2's "pushes no code side" and its criterion that the old owner's next transition leaves origin's code branch unchanged.
The record's own declared `PATTERN-pair-remote` text, "the code branch goes along after the records side", is breakable through it.

### [MEDIUM] D9's departure from the operator's intents basis needs the parent

**Location:** `_lym/discussion/decision-record.md` Open risks, D9 rationale.

**Issue:** The board entry's hub-stop-button member (operator, 2026-11-15) says the stop is built on the run FSM and its pause and cancel intents.
D9 writes no intent and rests its pushed state on D1.
No `Decision for adopt-build-resplit:` note and no added entry answer it, so the Open risks line stands and goes to the parent.

### [MEDIUM] The --take-hold bound overclaims the shell guard

**Location:** `_lym/discussion/decision-record.md` D6 Bound.

**Issue:** `internal/shellguard/doc.go:19-23` states that every row is a guardrail over static text and not a barrier, and that a written script or a quoted command substitution passes it.
"Only the operator's own shell runs it" is therefore false as a bound.
The real bound is already in D6: no check skipped, no run before a green check, and the old adopter stopping at its next re-read.

### [MEDIUM] The --take-hold guard row's own text reaches every agent

**Location:** `_lym/discussion/decision-record.md` D5 Bound, D6, Constraints.

**Issue:** A `shellguard.Row` carries `Steer`, the deny reason shown to the agent, and `Notice`, the system-prompt sentence announcing the guard (`internal/shellguard/rows.go:33-36`).
Every engine renders every row, so the new row's notice sits in every agent's prompt.
D5's "No refusal or notice an agent reads names `--take-hold`" and the matching constraint contradict a row that names it, and the record does not say what the row's texts are.

### [MEDIUM] A plain adopt re-run under its own adopt-fix hold resets the fix loop

**Location:** `_lym/discussion/decision-record.md` D6 steps 1 and 8, D5 upgrade.

**Issue:** Step 1 refuses only another machine's hold and then writes an `adopt` hold.
Only `--take-hold` keeps kind, fix task and try counts.
A re-run on the adopting machine, the very re-run D5's notice and D10 point at, therefore drops the fix task's exemption and resets the tries, breaking "within `adopt_fix_tries`".
Today's try count lives in a local tries file (`internal/adoptcli/fixloop.go:279`), which the upgrade's hold does not carry.

### [MEDIUM] A given-up code tip can still land

**Location:** `_lym/discussion/decision-record.md` D2 code-tip bullet.

**Issue:** The owing machine reads origin's last transition, then pushes the code tip on a separate ref.
The owner can give the tip up between the two, and nothing on the code ref refuses the later push, so "a given-up tip never lands" does not hold.

### [MEDIUM] The registry lets two machines claim one name

**Location:** `_lym/discussion/decision-record.md` D3.

**Issue:** `boardengine.OwnedElsewhere` is a plain string comparison today (`internal/boardengine/owners.go:175`).
D3's only refusal is a former name equal to another machine's current name.
A new machine named after another's former name, or two machines sharing a current name, both read an owner recorded under that name as their own, so "one identity" fails.

### [MEDIUM] D7's upgrade rests on a false premise about kept cancels

**Location:** `_lym/discussion/decision-record.md` D7 upgrade bullet.

**Issue:** `internal/battencli/cleanup.go:74` states that a cancelled run keeps its record while its pair stands.
`runfsm.Derive` reads `Cancelled` only when the record holds a cancel intent with a finish (`internal/runfsm/state.go:77-79`).
A legacy kept cancel therefore has a record and no intent: D7's rule skips it because a record exists, and the FSM reads it as neither cancelled nor kept.
Such a pair is neither listed nor torn down.
The bound-breaker lens rated this BLOCKING on the claim that D7 then tears the pair down.
That step does not follow from `state.go`, so the defect is a pair left unreported, not one destroyed.
It is a deferred research question the writer should have settled from the code.

### [MEDIUM] A hold freezes owed pushes, and the record does not say so

**Location:** `_lym/discussion/decision-record.md` D5 and D1.

**Issue:** `runfsm.Act` answers `wait` for `PushOwedOffline` first (`internal/runfsm/act.go:85-86`), so an owed run gets a `hold` slug entry.
D1's retry waits at the gate, so the owed transition cannot reach origin until the hold clears, and the adopter starts from the older transition.

### [MEDIUM] A handed-over pair lingers on a live dobby

**Location:** `_lym/discussion/decision-record.md` D2, D7, D4 step 4.

**Issue:** D2's hand-over ends processes and archives but keeps the pair.
D7 skips pairs owned elsewhere, and only D4's catch-up after an absence removes them.

### [MEDIUM] beam status loses its adopt section with nothing in its place

**Location:** `_lym/discussion/decision-record.md` D6 Retired list.

**Issue:** `internal/beamcli/cli.go:397` reads the adopt record into the status view's adopt section.
D6 retires the record and moves only the dobby's handover and the orch watcher onto the hold.

### [MEDIUM] No launcher for a machine that owns only beams

**Location:** `_lym/discussion/decision-record.md` D10 launcher bullet.

**Issue:** The launcher is written only at the prime dobby's start, and `dobbyengine.OwnedWorktrees` omits a prime another machine owns (`internal/dobbyengine/start.go:105`).
A machine owning only a beam never gets the launcher the entry asks for.

### [LOW] D8's departure from "every reader moves" goes to nobody

**Location:** `_lym/discussion/decision-record.md` D8 closing paragraph.

**Issue:** The entry's prime-is-root-beam member names the orch role, the dobby and the board verbs as readers that move to the named root.
D8 keeps the dobby's and board verbs' reads on the empty slug on purpose.
The choice may be sound, since `boardengine.ParentBeam` stays internal, but it overrides operator text and is not in Open risks.

### [LOW] The already-read takeover mark is unnamed

**Location:** `_lym/discussion/decision-record.md` D2 bullet 2.

**Issue:** "That this machine has not yet read" needs a per-machine record no decision places or writes.

### [LOW] D4's hand-over skips D2's tags and notice

**Location:** `_lym/discussion/decision-record.md` D4 step 4.

**Issue:** Step 4 archives as D2 does but omits the archive-tag push and the notice of a dropped intent, which D2's bound promises.

### [LOW] The upgrade notice omits the resume of its pause intents

**Location:** `_lym/discussion/decision-record.md` D5 upgrade bullet.

**Issue:** Runs whose entries became pause intents still read `paused` after the green check (`internal/runfsm/act.go:97-98`), and the notice names only the adopt re-run.

### [LOW] A lock-owed push is read as offline

**Location:** `_lym/discussion/decision-record.md` D1 teardown-lock bullet.

**Issue:** `PushOwedOffline`'s refusal says origin is unreachable (`internal/runfsm/act.go:115`), which misreads a push owed on a held teardown lock.

### [LOW] A Finalize cut after its local merge waits out a hold

**Location:** `_lym/discussion/decision-record.md` D2, D5.

**Issue:** `PastParentMerge` reads origin's parent branch (`internal/runfsm/act.go:29-30, 91`), so a merge that is only local does not get the landing's exemption from the hold.

### [LOW] The new hold caller collides with HoldCaller

**Location:** `_lym/discussion/decision-record.md` D5.

**Issue:** `pauserecord.HoldCaller` is the legacy `"adopt-hold"` caller (`internal/pauserecord/record.go:29-32`), which D5 keeps declared beside a new caller it calls `hold`.

## Rejected

- Bound-breaker F5's teardown counterexample: a legacy cancelled record with no cancel intent does not derive `Cancelled` (`internal/runfsm/state.go:77-79`), so D7 does not tear its pair down; the finding is kept as F8 at MEDIUM on the false premise and the unreported pair.
- Focus item on task filing: not a defect. D6's fix task is filed by `lym adopt` in the operator's process and by the fix task's `on_land` action run by the dobby, through the landed fix loop (`internal/adoptcli/fixloop.go:279`). The board refuses only a run agent's strand (`internal/boardengine/taskguard.go:14-16`), and no Decision files a follow-up of this run as a task.
- Focus item on the board entry: not a defect. No Decision changes this task's own entry; D8 changes board printing code, which the rubric allows.
