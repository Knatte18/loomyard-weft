---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D1 act rule, D4 second bullet, D8 step 3
    summary: After a takeover, D4 kills the old owner's landing past its parent-side merge while D1 lets the adopter drive the same Finalize, so the carve-out holds on neither side
    fix: >-
      Make one machine finish a landing past its parent-side merge after a takeover, and state it in D1, D4, D6 step 4, D8 step 3 and D9 alike.
      Recommended: the adopter finishes it, since D2 puts every transition on origin and a Finalize past its merge resumes from there.
      Then drop "and when another machine owns it" from D1's landing `act` line,
      drop the landing carve-outs of D6 step 4 and D9 in favour of D4's hand-over,
      and reword the `PATTERN-owner-machine` constraint and its declared change to say the owner finishes it.
      If the old owner finishes it instead, D8 step 3 writes no takeover entry for such a run,
      and D1 answers `wait` on the new owner until the batten record reads `landed`.
      Add an acceptance criterion: after a takeover, a landing past its parent-side merge is driven on exactly one machine.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D2 first bullet, D4 second bullet and Bound
    summary: A pair push sends the code side before the records side, and the takeover entry is on records branches only, so the old owner's post-takeover code commits land on origin's code branch
    fix: >-
      In D2, order a task pair's transition push records side first, and push the code side only once the records side has landed.
      In D4, a records push rejected over a takeover entry pushes no code side; its code commits go to the archive tag with the rest.
      State in D4 how the adopter's code push reads a code branch that still moved, and add an acceptance criterion:
      after a takeover, the old owner's next transition leaves origin's code branch unchanged.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D6 step 4
    summary: D6 step 4 removes the local pair as fabric's removal under an owned-elsewhere verdict does, and no such removal exists
    fix: >-
      Replace the precedent in D6 step 4 with the route the design adds:
      a removal through `internal/pairteardown` that ends the slug's processes, archives the local tips,
      deletes no origin branch and settles no board claim.
      Name it as a new mode of that composite, so `PATTERN-pair-remote`'s teardown rule still holds.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D1 states, D3 cancel intent bullet, D11 first bullet
    summary: The record never says which shed state a cancelled batten record carries, ignores the dobby's done-run clear and lym batten cleanup, and leaves a landed run whose record went with its pair in no D1 state
    fix: >-
      In D3, state the shed state a finished cancel leaves in the batten record and which remover may clear it:
      the dobby's done-run clear and `lym batten cleanup` must neither delete a kept cancel's task branch nor refuse a finished cancel forever.
      Say whether "the batten record goes with the pair's teardown" covers landed runs.
      In D1, add what a slug reads once its record is gone with a `done` board status, or state that it is no run and the act rule is never asked for it.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D7 upgrade bullet
    summary: At the upgrade a standing adopt pause entry, which every past adopt leaves, writes a hub-wide Bolt hold that stops every machine with no notice or way forward
    fix: >-
      In D7's upgrade bullet, either write no Bolt hold and only convert the entries to run pause intents,
      or have the upgrade post one notice to the operator naming the hold, its machine,
      and a re-run of `lym adopt` on that machine as the way forward.
      Add an acceptance criterion for the chosen behaviour.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D10 third bullet and Bound
    summary: A mirror runs at most once per interval with no trailing mirror, so on a normal move with the old orch idle the turns since the last mirror never travel, which the bound calls a crash loss only
    fix: >-
      In D10, add a trailing mirror: a turn end that falls inside the interval schedules one mirror at the interval's end.
      Reword the bound to cover a machine that stops or is taken over between mirrors, not only a crash.
      Add an acceptance criterion: a turn end inside the interval reaches origin once the interval ends.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D10 third and fourth bullets and Bound
    summary: The lease retry makes the old owner's late mirror win, and the adopter's next orch start then restores that stale mirror over its own newer local transcript
    fix: >-
      In D10, a mirror whose lease is lost to a mirror naming another machine stops and writes nothing,
      and the restore at an orch start keeps the local transcript when the record of D10's first bullet names this machine as the last launcher or resumer.
      Add an acceptance criterion: an old owner's late mirror never replaces the new owner's local transcript at its next start.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D2 second bullet, D3 pause intent bullet, D7 third bullet
    summary: A pause or hold that lands between a transition's commit and its push holds that push at the network gate, so the transition stays local for the whole pause
    fix: >-
      In D2, state that a transition's synchronous push passes the pause record's network gate, as `pauserecord.Admit` lets a lock holder through,
      so a pause or hold stops the run's next producer and never the push of a transition already committed.
      Add an acceptance criterion: a pause written between a transition's commit and its push leaves that transition on origin.
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D4 last bullet, D8 release paragraph
    summary: An adopter whose takeover push is rejected after a release and a second adopt rewrites its own takeover entry over the second adopter's, which then hands over, leaving no machine on the runs
    fix: >-
      In D4's last bullet, the adopter re-reads its hold and the Bolt owner before it writes the takeover entry again,
      and stops, naming the release, when either names another machine.
      Add an acceptance criterion for a release and a second adopt landing between an adopter's step 2 and step 3.
  - id: F10
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D8 steps 2 to 4, D6 steps 2 and 3
    summary: When the old owner pushes a transition between adopt steps 2 and 3, the adopter's local branches stay behind origin, and neither the takeover retry nor the catch-up moves them
    fix: >-
      In D8, after step 3 lands, fast-forward each moved run's local branches to origin's tips, which the takeover entry now heads.
      Or add to D6 step 3 that a run owned here whose local branch is behind origin is fast-forwarded.
      Add an acceptance criterion for a transition the old owner pushed between the adopter's fetch and its takeover.
  - id: F11
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md D10 Facts this rests on, second bullet
    summary: The evidence for resuming a transcript restored from another machine shows only same-machine resumes, so the cross-machine resume D10 rests on is unverified
    fix: >-
      Do the research and record the result under D10's Facts with its source and Claude Code version:
      copy a session's transcript into the project directory of another worktree path, run `claude --resume <id>` there,
      and record whether the session resumes with its history and its id kept.
      If it does not, change D10's restore to what the result allows.
  - id: F12
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/discussion/decision-record.md Open risks first bullet, D10
    summary: D10 pushes the orch's whole conversation to the weft repository, which is public, and only the operator can allow that
    fix: >-
      Parent asks the operator whether orch transcripts may go to the public weft repository.
      On a no, D10 names another place, such as a private repository, or drops the mirror; on a yes, the Open risks line goes.
  - id: F13
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/discussion/decision-record.md Open risks third bullet, D8
    summary: lym adopt --release lets the hub go on with failing tests, against the operator's decision that nothing happens until every failing test adopt found is fixed
    fix: >-
      Parent asks the operator whether `lym adopt --release` may exist.
      On a no, D8 drops it and states how a hold whose adopter stopped is cleared; on a yes, the Open risks line goes.
  - id: F14
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/discussion/decision-record.md Open risks fourth bullet, D7 and D8
    summary: D8 keeps the orch up and lets it cancel runs during a hold, against the operator's decision of no orch action until adopt is done
    fix: >-
      Parent asks the operator whether the orch may be up during a hold.
      On a no, D8 starts the orch only once the hold clears and names who reads the fix task's halts meanwhile; on a yes, the Open risks line goes.
  - id: F15
    severity: LOW
    class: decision
    origin: handler
    location: _lym/discussion/decision-record.md Open risks second bullet, D2 Bound
    summary: D2 stops every run while origin is unreachable, and the operator has not said whether offline progress is wanted
    fix: >-
      Parent confirms with the operator that no run may act offline; on a yes, the Open risks line goes.
  - id: F16
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D11 first bullet
    summary: D11's teardown reads the act rule, which answers wait for every terminal state, so the condition never fires for landed or cancelled runs
    fix: >-
      In D11's first bullet, name the condition as the FSM state `landed`, or `cancelled` without keep,
      with no pause or hold covering the run and no other machine owning it, instead of the act rule's answer.
  - id: F17
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D3 owner refusal, D5 second bullet, Constraints refusal-way-forward line
    summary: The refusals D3 and D5 add name only the other machine, and the refusal constraint covers neither
    fix: >-
      Give D3's owner refusal the way forward of running the verb on the owner machine, or `lym adopt` here first.
      Give D5's clash refusal the way forward of `lym machine name <other>` on this machine.
      Extend the `PATTERN-refusal-way-forward` constraint line to the refusals of D3 and D5.
  - id: F18
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D7 Bound
    summary: The hold lag bound holds for the dobby's passes only, since lym loom resume and lym tender resume read the local Bolt on a worktree whose dobby is down
    fix: >-
      In D7, have every verb that asks the act rule pull Bolt first when it can reach origin,
      or narrow the bound to the dobby's passes and name the verbs that read the local Bolt.
  - id: F19
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D8 release paragraph and step 8
    summary: A release over a live adopt stops it before step 8, so no dobby is started for its subjects on the adopter
    fix: >-
      In D8, state that an adopt stopping on a release still runs `orchcli.BeamUp` for the subjects it already owns,
      or that its output names a re-run of `lym adopt` as the way forward.
  - id: F20
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/support-log.md Question ledger Q5
    summary: Q5 lists a rejected push as an absence, while D6 lists a hand pass and no rejected push
    fix: >-
      Reword Q5 to the absences D6 lists: an observed sleep, a dobby start and a hand pass.
---

### [BLOCKING] A landing past its parent-side merge after a takeover

**Location:** `_lym/discussion/decision-record.md` D1 act rule, D4 second bullet, D8 step 3.

**Issue:** D8 step 3 writes the takeover entry on every moved run, a landing past its parent-side merge included.
On the old owner, that Finalize's next records push is rejected, and D4 hands the run over with no carve-out: it ends the processes and pushes nothing.
D1, D6 step 4, D9 and the `PATTERN-owner-machine` constraint all say that landing finishes there.
On the adopter, D1 answers `act` too, because it owns the run, and D7 holds no landing.
D8 retires the adopt pause that kept the adopter off it today.
So either the landing dies on the old owner while the record says it finishes, or both machines drive one Finalize.
The parent lock is machine-local, so it cannot keep two machines apart (state map 3.2).
Today's carve-out reads git state (`battenshed.FinalizePastMerge`, `fabricengine` `finalizeEntered`), so either machine can judge it after a fetch, and choosing one is the writer's call.

### [BLOCKING] Code commits land after the takeover

**Location:** `_lym/discussion/decision-record.md` D2 first bullet, D4 second bullet and Bound.

**Issue:** `PushPairAnchored` pushes the code side first, and attempts the records side even when the code side failed (`internal/fabricengine/pushanchored.go:126-128`).
The takeover entry sits only on records branches.
So the old owner's first transition after the takeover fast-forwards origin's code branch, and only then is its records push rejected.
D4's bound, "archived under a tag, not lost and not on the branch", breaks.
The adopter, reset to origin's code tip at D8 step 2, then has its next code push rejected over a branch that carries no takeover to read.

### [MEDIUM] D6 step 4 cites a removal that does not exist

**Location:** `_lym/discussion/decision-record.md` D6 step 4.

**Issue:** The step removes the local pair "as fabric's removal under an owned-elsewhere verdict does".
Fabric's reconcile never hands an owned_elsewhere pair to the teardown seam (`internal/fabricengine/doc.go:162`).
`internal/pairteardown`, the one teardown route under `PATTERN-pair-remote`, settles the board claim unless told not to, and its archive step deletes the weft branch on origin.
So the local-only removal D6 needs has no route, and the plan writer would follow a precedent that is not there.

### [MEDIUM] The cancelled and landed batten record against its existing removers

**Location:** `_lym/discussion/decision-record.md` D1 states, D3 cancel intent bullet, D11 first bullet.

**Issue:** The dobby's done-run clear removes any `done` batten record and deletes the task branch on origin (`internal/battencli/cleanup.go` `ClearDoneRun`).
`lym batten cleanup` removes only `done` records and refuses the rest.
D3 keeps a cancelled run's record but never says which shed state its finish leaves.
If that state is `done`, the dobby clears a kept cancel and deletes its branch.
Any other state leaves cleanup refusing it forever.
D3 also says the record "goes with the pair's teardown, D11's", and D11 tears down landed pairs.
A landed slug then has a `done` board status and no record, which matches none of D1's nine states.

### [MEDIUM] The upgrade writes a silent hub-wide hold

**Location:** `_lym/discussion/decision-record.md` D7 upgrade bullet.

**Issue:** Adopt pause entries lift only by hand (`internal/pauserecord/doc.go`), so every machine that has adopted holds them.
The first read after the upgrade then writes a Bolt `adopt` hold, which answers `wait` for every run on every machine.
The record names no notice for it.
Its clears are a green re-run of `lym adopt` on that machine or `--release`, which no agent-read text may name.
Another machine's `lym adopt` refuses, naming the machine, and nothing tells the operator why the hub stopped.

### [MEDIUM] A normal move loses the orch's last turns

**Location:** `_lym/discussion/decision-record.md` D10 third bullet and Bound.

**Issue:** A turn end mirrors at most once per mirror interval, and nothing mirrors a turn the interval skipped.
In the usual move, the old machine's orch goes idle a few minutes after a mirror and the operator adopts elsewhere.
The old machine's next mirror chance is its stop, which `hand_over` blocks, and D9 then ends the orch.
The adopter restores a mirror without those turns, which misses D10's purpose of a continuation.
The bound names only a crash.

### [MEDIUM] The old owner's late mirror wins

**Location:** `_lym/discussion/decision-record.md` D10 third and fourth bullets and Bound.

**Issue:** A mirror whose lease is lost re-reads the ref and pushes again, gated only by the act rule.
The act rule reads a local Bolt up to the dobby's pull interval old (`internal/dobbyengine/handover.go`).
So the old owner's late mirror replaces the adopter's.
If the adopter's orch is idle and then restarts, for example after a hibernate, its start sees a mirror naming another machine.
It restores that stale copy over its own newer transcript, and keeps its own only beside it.
The bound "the owner's next mirror replaces it" assumes a next mirror that may never come before that start.

### [MEDIUM] The network gate holds a transition's push

**Location:** `_lym/discussion/decision-record.md` D2 second bullet, D3 pause intent bullet, D7 third bullet.

**Issue:** Every remote git call of a run's process waits at the pause record's network gate (`internal/gitexec/gitexec.go:254`).
D3 and D7 keep enforcing pauses and holds there, and D2 drops fabric's pause deferral for a transition's push.
D2 says nothing about the gate itself.
A pause written between a transition's commit and its push holds that push for the whole pause.
That is when the operator stops a machine to go on elsewhere, so the transition never reaches origin.

### [MEDIUM] A release and a second adopt race the takeover retry

**Location:** `_lym/discussion/decision-record.md` D4 last bullet, D8 release paragraph.

**Issue:** An adopter re-reads its hold before each step, then gets its step-3 takeover push rejected.
Meanwhile `--release` and another machine's adopt have written that machine's hold, owner and takeover entries.
D4's retry writes the first adopter's entry onto origin's tip without re-reading Bolt.
The second adopter's next push is then rejected over a takeover naming another machine, so it hands over.
The first adopter stops at its next hold re-read, and no machine drives the runs.

### [MEDIUM] The adopter's branches stay behind origin

**Location:** `_lym/discussion/decision-record.md` D8 steps 2 to 4, D6 steps 2 and 3.

**Issue:** The old owner can push a transition between adopt step 2's fetch and reset and step 3's takeover.
It learns of the hold only at its next Bolt pull.
The takeover retry fixes only the records tip.
The catch-up only fetches, and pushes local commits ahead of origin.
Nothing fast-forwards a local branch that is behind, so the adopter resumes the run at origin's later transition over a stale worktree.

### [MEDIUM] Cross-machine resume is unverified

**Location:** `_lym/discussion/decision-record.md` D10, Facts this rests on, second bullet.

**Issue:** D10 rests on Claude Code resuming, under `claude --resume <id>`, a transcript written on another machine and copied into this worktree's project directory.
The cited evidence is lym's own stat check and today's resumes, which are all on the machine that wrote the transcript.
None of it shows that a transcript from another path, and maybe another operating system, resumes with its history.
The support log records that a deferred manual check was dropped in place of doing it, so the research is still owed.

### [MEDIUM] The orch's transcripts go to a public repository

**Location:** `_lym/discussion/decision-record.md` Open risks first bullet, D10.

**Issue:** The weft repository is public (D10's Facts), and the orch's conversation can carry anything the operator typed or the orch read.
Publishing it is the operator's call, and the record rightly raises it under Open risks, so it goes to the parent.

### [MEDIUM] `--release` against the operator's decision

**Location:** `_lym/discussion/decision-record.md` Open risks third bullet, D8.

**Issue:** The board's operator decision says nothing happens in the hub until adopt is done and every failing test it found is fixed.
`lym adopt --release` lets the hub go on without that, so only the operator can allow it.

### [MEDIUM] The orch up during a hold

**Location:** `_lym/discussion/decision-record.md` Open risks fourth bullet, D7 and D8.

**Issue:** The board's operator decision lists "no orch action" beside "no run starts".
D8 brings the orch up during a hold, and D7 lets the operator's `lym batten cancel`, which the orch can also run, go on.
Only the operator can settle that reading.

### [LOW] No offline progress

**Location:** `_lym/discussion/decision-record.md` Open risks second bullet, D2 Bound.

**Issue:** D2 stops every run while origin is unreachable.
The operator's requirement of the whole state on GitHub at all times points that way, but the operator has not said so, and the record asks the parent.

### [LOW] D11's teardown condition reads the act rule

**Location:** `_lym/discussion/decision-record.md` D11 first bullet.

**Issue:** D1 answers `wait` for every terminal state, and `landed` and `cancelled` are terminal.
A teardown gated on the act rule's answer therefore never runs.
Today's condition reads the run's coverage and owner directly (`internal/dobbyengine/ended.go`).

### [LOW] Refusals with no way forward

**Location:** `_lym/discussion/decision-record.md` D3 owner refusal, D5 second bullet, Constraints refusal-way-forward line.

**Issue:** D3's refusal of a pause or cancel of a run another machine owns, and D5's refusal of a clashing registration, name only the machine.
D5's refusal also fails D8 step 1's owner write.
The constraint line covers only the refusals of the act rule, the holds and D9.

### [LOW] The hold lag bound holds for the dobby only

**Location:** `_lym/discussion/decision-record.md` D7 Bound.

**Issue:** The dobby's probe tick is the only Bolt pull outside a catch-up (`internal/dobbyengine/handover.go`).
`lym loom resume` and `lym tender resume <slug>` read the local Bolt.
On a worktree whose dobby is down, they ask the act rule over a Bolt without the hold for as long as it stays stale.

### [LOW] A release mid-adopt leaves the subjects without a dobby

**Location:** `_lym/discussion/decision-record.md` D8 release paragraph and step 8.

**Issue:** An adopt stops at its next hold re-read once released, before step 8's `orchcli.BeamUp`.
Its subjects are owned here with no dobby to drive them, which misses "the runs go on by the act rule, with no hand lift".

### [LOW] The ledger's absences disagree with D6

**Location:** `_lym/discussion/support-log.md` Question ledger Q5.

**Issue:** Q5 picks "an observed sleep, a dobby start and a rejected push" as absences.
D6 lists a hand pass in place of a rejected push, which D4 handles instead.

## Rejected

- Walkthrough 6, a pause intent lost to shed's persist of the same `status.json`: `shedverbs/pause.go` already writes `pause_requested` into that file under its status lock through `state.UpdateJSON`, so the precedent for a second locked writer exists.
- Existing-paths 1, the new `hold` caller beside the legacy `pauserecord.HoldCaller` constant: the record names the values `hold` and `adopt-hold` apart, and choosing a constant's name is Plan's.
