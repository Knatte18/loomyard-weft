---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:206-211
    summary: "The origin reconcile fast-forwards and replays a pair's or the prime's weft past D10's stop and past a standing move record, so lyx orch start on a second machine applies another machine's re-anchor with no rewire"
    fix: "In D10 add a third stopping path beside the pull and the replaying push. The origin reconcile's weft move (reconcileRecordsBranch with ffWorktree and replayWeft, run hub-wide by lyx orch start, lyx adopt, lyx hibernate and lyx cleanup, and run-scoped on the run's own pair) applies the same test before it moves a pair's or the prime's weft. When the tip it would leave checked out, or the replay result, carries a marker other than the checked-out one (D5 value for a markerless tip), or the upstream has commits the local tip lacks while the two tips carry different markers, or the pair has a standing D7 move record, it leaves the branch unmoved and reports it stopped with the way forward lyx fabric anchor <slug> --set <anchor>, never refusing the orch start itself. Name the shared marker comparison once in fabricengine so the pull, the push and the reconcile call one function. Add D7 step 1's list of refusers the reconcile moves of the worktree's weft. Add to Acceptance criteria that after the prime's change on one machine a hub-scope origin reconcile on a second machine leaves the prime's weft and its .lyx unchanged and reports the way forward, and that no origin reconcile moves a weft whose pair has a standing move record. Correct D10's rationale so it lists every path that can change a checked-out marker."
  - id: F2
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:202-204
    summary: "The checkout refusal names lyx fabric anchor --set <target anchor> as its way forward, but D7 validation requires the target anchor to be a directory the current warp checkout tracks, which a diverged branch may lack before the checkout"
    fix: "In D10, after the sentence naming the --set way forward, add the case where the target's anchor is not a tracked directory of the worktree's current warp checkout. The refusal then names the way forward of updating the warp checkout by hand to a commit that tracks the target anchor, then lyx fabric anchor --set <target anchor>, then the checkout. Move the sentence beginning That cost is accepted because, through the clause ending with Add names no route around it, to support-log.md's Rejected alternatives as a rejected alternative, keeping in D10 only the requirement that the refusal names the --set and its cost to the branch being left."
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:84
    summary: "The integration reads the marker of each replayed commit's parent tree, a read D5's list of places using the markerless value does not name, so a local commit made before migration has no defined source anchor"
    fix: "In D5's list of places that write or compare an anchor, add a replayed local commit's parent tree. In D10's integration paragraph, change the phrase its paths under the anchor its parent commit's marker names to read its parent commit's marker, or D5's value of the parent's tree when that tree carries none."
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:120
    summary: "The move-lock refusal names the live run's pid, but the move record holds only A, B, the kind and U, and the file lock records no holder"
    fix: "In D7 step 1, state that the move record also holds the verb's pid and start time, written right after the move lock is taken, and that the move-lock refusal and the Windows live-dobby refusal read the pid from it. A re-run rewrites the record's pid with its own."
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:259
    summary: "The criterion for a standing move record lists lyx beam start starting no dobby but not that a live dobby starts no pass or task, which step 1's pass wait depends on"
    fix: "In the Acceptance criteria bullet on lyx fabric anchor --set, add that while the move record stands a live dobby of a covered worktree starts no pass and no task, and that dobbyengine.EnsureLive reports each covered dobby it did not start."
---

### [BLOCKING] The origin reconcile moves a weft past D10's stop and past the move record

**Location:** `_lyx/discussion/decision-record.md:206-211` (D10), `:131` (D7 step 1), `:262` and `:266` (Acceptance criteria)

**Issue:**
D10 stops a weft pull and a replaying weft push that would change the checked-out marker, and D10's rationale concludes that "checkout, pull, push and promote each refuse or stop before changing anything, so none leaves a half-rewired pair".
They are not the only paths that move a checked-out weft.
`internal/fabricengine/originreconcilemoves.go` holds the origin reconcile's moving half: `reconcileRecordsBranch` fast-forwards a records branch behind origin through `ffWorktree`, and replays a diverged one through `replayWeft`.
`moveAll` moves "the prime's records before its code, then each pair's", and `movePrime` moves the prime's branches with no live-pair test.
`ReconcileOrigin` with an empty `ReconcileScope` runs from `lyx orch start` (`internal/orchcli/start.go`, `reconcileHub`, whose package doc says the reconcile "can ... move branches"), `lyx adopt`, `lyx hibernate` and `lyx cleanup`.
Those moves consult neither D10's marker comparison nor a D7 move record.
They take the weft write lock by try-acquire only, so they proceed whenever the verb is not inside step 4.

Two consequences follow.
After the prime's change on machine 1, machine 2's first `lyx orch start` fast-forwards its `main-weft` to the commit that carries marker `B` and the content at `B`.
The checked-out marker now reads `B` while the junctions sit at `A`, and the never-tracked `A/.lyx` (reed state, gate slots, driver locks, dobby state) stays behind at `A`, which is the half-rewired pair D10 exists to prevent.
The acceptance criterion at `:262` ("a second machine's prime pull and push stop with both sides unchanged") passes while this path stays open.
Second, while a D7 move record stands, a hub-scope reconcile run from another worktree can fast-forward or replay the moving worktree's weft in steps 1 to 3 and 5 to 8, between the dobby end and the commit, which neither the record nor the parent lock prevents.

This is a bypass of a guard that the Decision introduces without stating what it lets through, so the attack-surface question applies.
It is a missing stopping path in the mechanism, not an incomplete enumeration of call sites.

**Focus settlements recorded here:**
The weft write lock being held only across step 4 is sound: taking it from the record's write through the pass wait would deadlock against a Finalize that holds the parent lock and needs the weft write lock for its parent-records commit.
The parent lock is held from its acquisition through step 8, and the criterion at `:260` states holding ("holds the parent lock of the worktree's branch until its last step").
The writers that could cross steps 3 and 5 to 8 without either lock are the origin reconcile moves above; closing them by the record check in the fix leaves no unnamed writer except the ones D7 already leaves to the operator.

### [LOW] The checkout refusal's way forward can be a refused command, and carries deliberation narrative

**Location:** `_lyx/discussion/decision-record.md:202-204` (D10)

**Issue:**
The refusal names `lyx fabric anchor --set <target anchor>` and then the checkout.
D7's validation requires `B` to be "a directory git tracks in the worktree's warp checkout", and the warp has not yet switched to the target branch.
A target branch that added the anchor directory after the current branch diverged leaves the directory untracked in the current checkout, so `--set` refuses it, and the only way forward D7 names for an untracked `B` covers a directory the warp's fetched upstream tracks, which the current branch's upstream does not.
The operator is sent in a circle.

The same paragraph holds deliberation: the sentence "That cost is accepted because ... `Add` names no route around it" justifies the choice and dismisses an alternative.
The requirement it supports, that the refusal names the `--set` with its cost to the branch being left, is already stated in the sentence before it, so the completeness test passes and the narrative belongs in `support-log.md`'s Rejected alternatives.

### [LOW] A replayed commit's parent marker has no markerless rule

**Location:** `_lyx/discussion/decision-record.md:84` (D5), `:215` (D10)

**Issue:**
D5 defines one value for a markerless tree and lists where it applies: D8 migration, an adopt, D10's checkout, D10's pull and push comparison, and the integration's source anchor, `U`'s marker.
Every listed place uses it.
D10's integration also reads a marker per replayed local commit: "its paths under the anchor its parent commit's marker names re-rooted to `T`".
A local commit made before the branch was migrated, still unpushed, has a markerless parent tree, so "the marker it names" does not exist, and D5's list does not name this read.
D7's `A` is covered, since a markerless weft is refused with `lyx fabric reconcile`, and the pull and push comparison and the checkout are covered by name.

### [LOW] The move-lock refusal names a pid nothing records

**Location:** `_lyx/discussion/decision-record.md:120`, `:125-128`, `:122`

**Issue:**
The refusal for a held move lock says "the refusal names its pid".
Step 1 writes a move record holding `A`, `B` and the kind, plus `U` at step 4, and `internal/lock` (`AcquireWriteLock`, `TryAcquireWriteLock`) stores no holder.
The refusal has no source for the pid, so the plan writer must add one to the record without the Decision saying so.
The Windows live-dobby refusal does have a source, the run-process records.

### [LOW] The acceptance criterion omits the live dobby's pass gate

**Location:** `_lyx/discussion/decision-record.md:259`, `:129-131`

**Issue:**
Step 1 makes a live dobby start no pass and no task while the record stands, and the pass wait in step 1 and the end in step 2 rest on it.
The criterion lists `lyx beam start` starting no dobby, the reconcile refusal, the adopt refusal, the `Add` refusal and the pull and push refusal, which covers the refusals focus item 5 asks about, each with its way forward in D5, D7 and D8.
It does not list the gate on a dobby that is already live, so a test derived from the criteria can pass with the gate missing.

## Rejected

- Focus item 2, the pass wait: the wait covers the worktree's own dobby and, for the prime, every dobby of the machine, whatever run each drives, and reads a pass as in flight from the driver lock and from a live non-spared child of the dobby in the process table.
  `spawnPass` in `internal/beamcli/dobbydeps.go` starts the pass with `exec.Command` from the dobby, so the pass is a child of the dobby until the dobby ends, and it registers itself only after it starts, which is the case the process-table read exists for.
  Not a defect.
- Focus item 3, the not-landed recovery: `B/.lyx` is renamed back to `A/.lyx` before any reset or deletion, a state with both refuses, and the reset pathspecs and the untracked deletion exclude `.lyx` in the text.
  Not a defect.
- Focus item 4, the integration's landed test: `U` is written at step 4, after the pass wait and the parent lock and under the weft write lock, a weft pull is refused while the record stands, and a record with no `U` reads as never having reached step 4.
  An unlanded integration reads as landed only if `HEAD` already carries marker `B` and contains `U`, which the integration's own result alone satisfies.
  Not a defect.
- Focus item 5, the other starters: `lyx adopt` and `Add` each refuse before touching anything with a way forward naming `lyx fabric anchor <slug> --set <B>`, and the criterion at `:259` lists both.
  The only gap is the dobby gate in the last finding.
- Focus item 7, the integration conflict's way forward: it tags the local tip, saves the changes of the conflicting commit and of every later local commit outside the weft, resets to the conflicting commit's parent, and writes all of them back at `T`, so every later local commit is preserved.
  Not a defect.
