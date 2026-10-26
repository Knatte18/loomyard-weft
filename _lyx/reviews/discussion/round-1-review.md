---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D9 last bullet (line 137)
    summary: The pawl run holds the beam entry's run status for the beam's whole life, and D9 lifts the landing hold only for a closed list of readers, so boardengine's membership check (project.go:102) refuses every task joining the beam and loom peers lists the beam as an in-flight peer for weeks.
    fix: In D9's last bullet, replace the closed list with a rule over every reader of a beam project entry's run status. While the pawl run is at Beam-Await, or before it, its run status reads as no run hold to every board and batten reader. Membership (checkProjectField) accepts a task joining the project, the entry stays in the Project layer, the run lock leaves its fields editable, members fork, verbs run inside the beam and loom peers does not list it as a peer. Past Beam-Await, each reader refuses or locks as today's run hold does. Name membership in the bullet as one consequence, and add an acceptance criterion that a task upserted with project set to a beam at Beam-Await is accepted.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D10 (line 144) and D11 (line 162)
    summary: Each pawl run is driven by its parent's dobby and sits at Beam-Await for weeks, and dobby counts every run it drives, halted ones included, against max_runs (default 3), so three beams under root stop root's dobby starting any task.
    fix: Add a bullet to D9 (or D11) stating that a pawl run at Beam-Await takes no slot under its parent dobby's max_runs, and that it takes one again from the first row past Beam-Await. State the bound that the parent dobby still schedules passes for it. Add an acceptance criterion that a parent with max_runs pawl runs at Beam-Await still starts a ready task.
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D3 last bullet (line 56), D9 last bullet (lines 137-138), D11 (line 161)
    summary: The hold is defined three ways (at Beam-Await, past Beam-Await, and covering Worktree-Create), which leaves the rows before Beam-Await undefined. It is also never tied to the signal its readers judge, the board entry's state and producer status, whose write only warns on failure, so the hold can lag the run with no bound.
    fix: State the hold once in D9, over named pawl rows. The run holds no landing hold from Worktree-Create through Beam-Await, and holds one from Parent-Align on. Say it is read from the producer half of the board entry's run status. State the bound for a failed board write, for example that Beam-Await is not Done until the status naming the next row is written. Have D3's last bullet and D11 refer to that rule instead of restating it.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D9 third bullet (lines 131-132)
    summary: Beam-Await re-runs the land checks before it stops the beam's dobby and before the board status leaves Beam-Await. A member the still-live dobby or a hand batten run starts in that window forks from a beam that then aligns, reviews and lands, and no later row re-checks.
    fix: Reorder D9's accepted-request steps to stop the beam's dobby first, then re-run lyx beam land's checks, then write the snapshot, remove the request and be Done. On a failed check, restart the dobby and keep waiting. Alternatively, add a BeamLandRefusal re-check at Finalize that halts with the members named. Add the race to the acceptance criteria.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D6 (lines 97-98) and Acceptance criteria (line 220)
    summary: A beam's pawl run is resumed only by adopting its parent, so adopting root before G strands G's pawl run with no driver on either machine. The acceptance criterion also implies that adopting G resumes it when the parent is owned here, which D6 contradicts.
    fix: In D6, state that adopt resumes a beam's pawl run whenever, after the owner write, this machine owns both the beam and its parent, whichever of the two the adopt named. Reword the acceptance criterion to match, and add one for the order root then G.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D6 (line 99)
    summary: Adopt runs each adopted project's beam steps without D3's refusal of a beam past Beam-Await, so adopting a beam whose landing is in flight elsewhere starts a second dobby and orch that the remote Beam-Down never stops.
    fix: In D6, state that adopt skips the beam steps for a beam whose pawl run is past Beam-Await (the hold of F3), reports it with lyx batten status <slug>, and still writes the owner. Alternatively, refuse to adopt such a beam with that way forward.
  - id: F7
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lyx/discussion/decision-record.md D4 last bullet (line 70)
    summary: D4 rewrites every way forward that names lyx prime to name the shortcut lyx s. That contradicts the landed rule from commit fb5c10c3 that messages name the full command, never a shortcut.
    fix: Change D4's last bullet so that every way forward, notice, stencil and skill naming lyx orch start or lyx prime names lyx beam start. Cite fb5c10c3's rule that messages name the full command.
  - id: F8
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D6 (line 99) and D9 third bullet (line 132)
    summary: Adopt's StartDobbies (adoptcli/deps.go:122) and the anchor move (fabriccli/anchordeps.go:272) call dobbyengine.EnsureLive over every owned beam, so they restart a beam dobby that Beam-Await stopped, and D9 states no bound for that.
    fix: State in D1 that every EnsureLive caller (adopt, the anchor move and lyx beam start) starts only the beams it names. State in D9 that a beam past Beam-Await gets no dobby from any of them, or bound the churn by the refused-start backoff and say so.
  - id: F9
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D4 second bullet (line 64)
    summary: The phrase "runproc gains that role-scoped end if it lacks one" defers a checkable fact. runproc has End, EndSparing and EndProcess, and no role-scoped end.
    fix: Replace the clause with the answer. runproc has no role-scoped end (End, EndSparing and EndProcess only, runproc/end.go). D4 adds one that ends the live records of one role under an owner, with End's signal sequence and bound.
  - id: F10
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D5 fifth bullet (lines 82-83)
    summary: Batten's resume writes a task owner for a run-held task with no owner (resumeOwnerClaim). Under D5 that is every task, so if the claim is kept, every resume asks the narrowed setter for a task owner and is refused.
    fix: In D5's owner-setter bullet, state that batten's start claim and its resume claim both write no owner, and that the resume's owner write goes.
  - id: F11
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D7 (line 105)
    summary: D7 lets the watcher stop the orch on an owner read, but never says what an unreadable board or owner does.
    fix: Add to D7 that a failed owner read reads as not owned elsewhere and is logged once per episode, as the dobby's own owner read does, so the watcher stops the orch only on a read naming another machine.
  - id: F12
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D12 (lines 170-172)
    summary: The can-land check fires only when a run goes done or a cancel removes it, not when a member leaves the beam by a board edit or a hand done. The one-notice-per-episode memory also has no home that survives the dobby restarts D9 and lyx beam start cause.
    fix: In D12, have the dobby check BeamLandRefusal on every board change it already ticks on as well as on a run's end. Name the record that holds the sent-notice episode, for example a marker under the beam's dobby state, so a restarted dobby sends no second notice.
  - id: F13
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D9 third bullet (line 132)
    summary: A crash after Beam-Await removed the request or stopped the dobby, but before Done, re-enters Beam-Await with no request and no dobby, and no notice or way forward says so.
    fix: State in D9 that a re-entered Beam-Await finding the beam's dobby stopped restarts it, or posts the beam's orch a notice naming lyx beam land <slug> again. Order the steps so the request is removed last, just before Done.
  - id: F14
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D5 third bullet (line 79)
    summary: On a board with no root owner, D5 reads every entry but a recorded beam owner as owned by no other machine, which contradicts D5's own rule that a task reads its beam's owner.
    fix: Reword D5's sentence so that, on a board with no root owner, a recorded beam owner holds for the beam and every entry whose ParentOf answer is that beam, and every other entry reads as owned by no other machine.
  - id: F15
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D3 opening line (line 49)
    summary: D3 says its refusals come before the verb writes anything, but its owner refusal reads the Bolt the reconcile just pulled. D1 orders the reconcile and the root claim, both writes, before the refusals.
    fix: Reword D3's opening to say the refusals come before any beam step. Say the hub check, task-pair check and unknown-slug check come before the reconcile, and the owner, anchor-move and past-Beam-Await refusals come after the reconcile and claim, before any dobby, window or orch start.
  - id: F16
    severity: LOW
    class: consistency
    origin: lens:goal-scope
    location: _lyx/discussion/decision-record.md D6 (line 91)
    summary: The entry's line There is no adopt of a beam is neither met nor interpreted, while D6 takes beam project slugs as adopt's arguments.
    fix: "Add one sentence to D6 reading the entry's line. A beam has no adopt of its own: lyx adopt names the beam's project, and the pair, dobby and orch move with it."
  - id: F17
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/discussion/decision-record.md Acceptance criteria (lines 209-234)
    summary: Several bounded decisions have no must-cover scenario. Missing are D2's fallback when the provider transcript is gone, D12's re-arm after a member opens again, and D10's create refusals (a beam owned elsewhere, and a parent owned elsewhere).
    fix: "Add acceptance criteria: a recorded session whose transcript is gone starts fresh from the note and names it; after a can-land notice, opening a member and landing it again sends a second notice; lyx beam create refuses a beam another machine owns naming lyx adopt <slug>, and a parent another machine owns naming lyx adopt <parent>."
  - id: F18
    severity: NIT
    class: consistency
    origin: handler
    location: _lyx/discussion/decision-record.md D11 last bullet (line 166)
    summary: D11's last bullet carries deliberation (why two machines' records do not collide) beyond its requirement that batten run records stay where they are.
    fix: Keep in D11 only that every run's batten record stays under its own slug's directory in the prime's records, a beam's members' included. Move the justification (different paths, the records push's replay merges) to support-log.md's Turn 7 walkthrough-1 line.
---

### [BLOCKING] A beam can take no new member while its pawl run holds it

**Location:** `decision-record.md` D9, last bullet

**Issue:** From the batten claim (`running · Worktree-Create`, `internal/battencli/owner.go:195`) onward, the pawl driver writes `<state> · <producer>` onto the beam entry (`internal/loomcli/wiring.go:200-216`), so the entry is run-held for the beam's whole life.
`checkProjectField` refuses any task whose `project` names a run-held project ("is held by a run", `internal/boardengine/project.go:102`).
D9 lifts the hold only for a closed list (fork precondition, verbs, start, Project layer, run lock), so `lyx board upsert {"slug":"t1","project":"G"}` is refused, and the beam can never grow.
Loom peers also treats every run-held entry as an in-flight peer (`internal/loomcli/peers.go:99-108,196`).
Both come from one cause: the rule is a list of readers instead of a rule over every reader.
Three lenses found this independently.

### [BLOCKING] Pawl runs fill the parent dobby's max_runs

**Location:** `decision-record.md` D10, D11

**Issue:** The parent's dobby drives the pawl run (D10).
Dobby starts a task only "while fewer than `max_runs` runs it drives are in flight: halted runs count" (`internal/dobbyengine/doc.go:58`), and the default is 3 (`internal/dobbyengine/template.yaml:1`).
Three beams parked at Beam-Await stop root's dobby from starting any task, and a nested beam does the same to its parent beam.
No decision exempts a run at Beam-Await from the cap or states the effect.

### [MEDIUM] The landing hold is defined three ways and never tied to its signal

**Location:** `decision-record.md` D3 last bullet, D9 last bullet, D11

**Issue:** D3 and D9's last sentence say "past Beam-Await", D9 says "at Beam-Await", and D11 relies on the hold covering Worktree-Create.
Worktree-Create, Beam-Up and a halt at Beam-Up are neither "at" nor "past" Beam-Await.
Every hold reader (`beamLanding`, `internal/battencli/refusal.go:96-104`; `beamForkPrecondition`, `internal/battencli/beamfork.go:62`; the run lock and the layers) judges the board entry's status.
The driver writes that status warn-only (`internal/loomcli/wiring.go:226-235`), so a failed write leaves `running · Beam-Await` standing through the landing, with no bound stated.

### [MEDIUM] Members can fork in the window after the land re-check

**Location:** `decision-record.md` D9, third bullet

**Issue:** The order is: re-check, snapshot, remove the request, stop the dobby, Done.
Until the dobby stops and the board status leaves Beam-Await, the live dobby or a hand `lyx batten run <member>` can start a member, which passes `beamForkPrecondition` and forks from G.
No row after Beam-Await re-runs `BeamLandRefusal`, so G aligns, reviews, lands and tears down with a member forked from it.

### [MEDIUM] Adopting root before the beam strands the beam's pawl run

**Location:** `decision-record.md` D6, lines 97-98; acceptance criterion line 220

**Issue:** Root and G are on Neve.
1. `lyx adopt root` on OSL-1046 does not resume G's pawl run, which reads as owned by Neve.
2. Neve's prime dobby then exits, because root is now owned elsewhere (`internal/dobbyengine/loop.go:208`), so nothing on Neve drives or pauses the run.
3. `lyx adopt G` on OSL-1046 resumes only runs whose parent is G's worktree, and D11 says dobby never starts a beam project.

The run has no driver anywhere, and `lyx beam land G` writes a request nothing consumes.
The entry's example order (G, then root) works.
The acceptance criterion's "while another machine owns the parent" implies resumption when the parent is owned here, which D6 forbids.
A hand pass remains as a way out (D11), but nothing names it, so this is MEDIUM rather than BLOCKING.

### [MEDIUM] Adopt starts a beam's parts while its landing is in flight elsewhere

**Location:** `decision-record.md` D6, line 99

**Issue:** D1 defines the beam steps without D3's refusals, and D6 runs them for every adopted project.
Adopting G while Neve's pawl run is in Parent-Align, the perch or Finalize starts a dobby and an orch on OSL-1046.
Neve's Beam-Down stops only Neve's parts, so those keep running on a landed beam.

### [MEDIUM] Ways forward would name a shortcut against a landed rule

**Location:** `decision-record.md` D4, last bullet

**Issue:** The rule landed in `fb5c10c3` (Design decisions): "Messages name the full command, not the one-letter shortcut. The full command exists independent of the shortcut file and is greppable; a bare `lyx r` in a refusal reads as a typo."
D4 tells every way forward naming `lyx prime` to name `lyx s` instead.
The writer can settle this by citing that rule and naming `lyx beam start`.

### [LOW] Other EnsureLive callers restart a beam dobby Beam-Await stopped

**Location:** `decision-record.md` D6, D9

**Issue:** `internal/adoptcli/deps.go:122` and `internal/fabriccli/anchordeps.go:272` call `dobbyengine.EnsureLive(hubgeom.DobbyStart(prime))`, which starts every owned beam's dobby.
D1 narrows only `lyx beam start`, so an adopt of an unrelated project during G's landing restarts G's dobby, and the members it starts are refused at Worktree-Create, over and over.

### [LOW] D4 defers a fact about runproc that can be answered now

**Location:** `decision-record.md` D4, second bullet

**Issue:** `internal/runproc/end.go:56-70` offers `End`, `EndSparing` and `EndProcess`, and no role-scoped end.
The dobby is registered under the beam's slug (`internal/beamcli/dobbydeps.go:65`), as is the pawl run's loop (`internal/battencli/arm.go:525`).
The record should state that the role-scoped end is new.

### [LOW] Batten's resume owner claim is left in place

**Location:** `decision-record.md` D5, fifth bullet

**Issue:** `resumeOwnerClaim` (`internal/battencli/owner.go:241-271`) writes a task owner when a run-held task has none.
D5 removes the start claim's owner write but not this one, which the narrowed setter would refuse on every resume.

### [LOW] The watcher's owner read has no failure direction

**Location:** `decision-record.md` D7

**Issue:** The dobby reads an unreadable owner as not owned elsewhere (`internal/dobbyengine/loop.go:236-249`).
D7 gives the watcher a stop action without that direction.

### [LOW] The can-land trigger and episode memory are incomplete

**Location:** `decision-record.md` D12

**Issue:** A member moved off the project or marked done by hand never triggers the check.
The episode memory's home is unnamed, while D9 and `lyx beam start` restart the dobby.

### [LOW] A crash in Beam-Await's accept leaves the beam silently stuck

**Location:** `decision-record.md` D9, third bullet

**Issue:** If the driver dies after removing the request or stopping the dobby, the re-spawned driver waits with no request and no dobby, and nothing tells the orch.

### [LOW] D5's no-root-owner sentence contradicts members following their beam

**Location:** `decision-record.md` D5, `Owners.Effective` bullet

**Issue:** "every other entry as owned by no other machine" overrides the bullet before it, so a task under a recorded beam owner reads as unowned on a board with no root owner.

### [LOW] D3's "before it writes anything" contradicts D1's order

**Location:** `decision-record.md` D3, opening

**Issue:** The owner refusal reads the Bolt the reconcile pulled, and D1 runs the reconcile and the root claim, both writes, first.

### [LOW] The entry's "There is no adopt of a beam" is not read

**Location:** `decision-record.md` D6

**Issue:** D6, D3 and D10 name `lyx adopt <slug>` for a beam, and the record never says how that squares with the entry's line.

### [LOW] Acceptance criteria miss some bounded decisions

**Location:** `decision-record.md` Acceptance criteria

**Issue:** D2's transcript-gone fallback, D12's re-arm, and D10's two ownership refusals for create have no must-cover scenario.

### [NIT] D11 carries deliberation about batten records

**Location:** `decision-record.md` D11, last bullet

**Issue:** The requirement is that records stay put, and the rest argues why that is safe.
The completeness-before-leanness test keeps the requirement in D11 and moves only the argument.

## Rejected

- Tender's slug-wide `runproc.End` reaching the beam's dobby (handler check): it only runs for an owner pause of an entry owned elsewhere, where the dobby exits anyway (`internal/tendercli/deps.go:321`, `internal/dobbyengine/loop.go:208`).
- D1's `EnsureLive` "for that worktree only" having no single-worktree mode (existing-paths, survives): D1 states the narrowing, and the signature is plan work.
- D11's claim that a member-free cancel ends the orch: it holds, because `stopPairStrands` stops the pair's strands (`internal/pairteardown/cancel.go:214`).
