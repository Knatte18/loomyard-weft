---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D7 step 1 (parent-lock wait) and steps 2-8
    summary: D7 samples the parent lock once and holds neither it nor the weft write lock, so a child loom run's Finalize can commit parent records into, and land into, the worktree mid-move
    fix: In D7 step 1, after writing the move record, have the verb acquire the parent lock of the worktree's branch through internal/parentlock as its own owner and hold it until step 8, so a child's Finalize queues behind it; also hold the weft write lock (.weft/weft.write.lock) across step 4. State this bound in D7, and change the acceptance criterion to say the verb holds the parent lock rather than waits for it.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D7 recovery, Not landed bullet
    summary: The Not-landed recovery deletes what HEAD does not track under B/<names> before renaming B/.lyx back, and .lyx is a wired name, so it deletes the worktree's machine-local .lyx
    fix: In D7's Not-landed bullet, rename B/.lyx back to A/.lyx (or refuse when both exist) first, and exclude .lyx from the B/<names> pathspecs the reset and the untracked deletion act on; say that .lyx is never reset or deleted by the recovery.
  - id: F3
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D7 refusals (prime) and steps 1-2
    summary: Step 2 can kill a pass in flight that the refusals and step 1's wait never saw, for any beam dobby on a prime change, and on any worktree for a pass spawned before the record but not yet holding its driver lock
    fix: In D7 step 1, make the wait cover every run of every dobby step 2 will end (for the prime, every dobby of this machine, not only runs parented to the prime), and read a pass as in flight when its dobby has a live child process other than a spared push or sync, from the process table, as well as when its driver lock is held; step 2 ends a dobby only once that reads none.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D7, paragraph on the move record and EnsureLive callers
    summary: lyx adopt is bounded only on dobby starts; while a move record stands it still runs Topology.Add, StartFirstPass resumes, lyx loom resume and the finalize gate suite, with no stated bound
    fix: Add to D7 that lyx adopt refuses, before step 1 of its own, while a move record stands in the prime or in any worktree whose runs it would resume or add a pair under, naming a change in progress while the move lock is held and lyx fabric anchor <slug> --set <B> once it is free; add the refusal to the acceptance criteria.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D5 (Add's anchor) and D7 step 4
    summary: Add from a worktree with a standing move record is not refused, and after a kill between the marker write and the commit it gives the new pair anchor B while its forked content and marker sit at A
    fix: Add to D5 that Add (lyx fabric add, lyx beam create, adopt's Add) refuses a fork point whose weft has a standing move record, before touching anything, naming lyx fabric anchor <slug> --set <B> from the record; add it to the acceptance criteria.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D7 step 1 record and recovery landed test for an integration
    summary: The integration landed test compares HEAD with a tip recorded before step 1's wait, so any weft commit or fast-forward meanwhile makes an unlanded integration read as landed
    fix: Record the upstream tip U the integration rebuilds on, written at the start of step 4 after the wait, and define an integration as landed when HEAD's committed marker is B and U is an ancestor of HEAD; also have weft pulls refuse while a move record stands, naming the change in progress.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D7 prime refusals
    summary: The prime refusal of a live creel daemon rests on a false premise and has no way forward, since creel state is under <hub>/_board/.lyx/creel and no lyx verb stops the daemon
    fix: Drop the creel daemon from D7's prime refusals and from the acceptance criterion that lists the prime's refusals; keep the hibernate daemon refusal, naming lyx hibernate cancel as its way forward.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D10 pull and push stop, and D5's list of uses of the markerless value
    summary: D10's pull and push comparison and the integration's source anchor read a markerless tip as a different marker, so after D8's migration commit a pair whose upstream gained markerless commits stops on every pull and push, with no defined integration source
    fix: In D5, add D10's pull and push comparison and the integration's source anchor (U's marker) to the places that read a markerless tip at D5's value; in D10, state that a markerless tip compares by that value.
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D10 checkout refusal way forward
    summary: The checkout refusal's way forward re-anchors the branch being left, a branch other machines share, only to make a branch switch possible
    fix: Change D10's checkout way forward to name a step that leaves the current branch's anchor alone, such as lyx fabric add for a pair on the target branch, or state that --set then checkout re-anchors the branch being left and why that is acceptable; adjust the acceptance criterion to match.
  - id: F10
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D10 integration conflict way forward
    summary: The conflict way forward resets to the conflicting commit's parent, dropping every later local commit from the branch with nothing saving them
    fix: Change the D10 conflict way forward to save the changes of the conflicting commit and of every later local commit aside (or tag the local tip under archive/<slug>/<tip> before the reset), and to write all of them back at T after the re-run.
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D7 refusals and its sentence It skips no dirtiness check on the weft
    summary: D7 claims it skips no dirtiness check, yet no refusal checks the weft for uncommitted or untracked files under A/<names> on a first run
    fix: Add to D7's refusals a dirty weft, uncommitted or untracked paths under A/<names> or the marker, naming lyx fabric commit or removing the files, and drop or point the It skips no dirtiness check sentence at that refusal.
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D7 checked-out anchor A and D5 markerless value
    summary: D7 takes A from lyxcwd's fallback for a markerless worktree, while D5 says a markerless branch's anchor, wherever one is written or compared, is the shallowest directory holding a wired name
    fix: In D7, refuse a worktree whose weft has no committed marker, naming lyx fabric reconcile first, or add D7 to D5's list of uses.
  - id: F13
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md acceptance criterion for lyx fabric anchor <slug> --set B
    summary: The acceptance criterion claims the verb spares pushes and board syncs, while D7 step 2 ends those an ended dobby spawned
    fix: Narrow the acceptance criterion to sparing the pushes and board syncs outside the ended dobbies' process trees.
---

### [BLOCKING] The move never holds the parent lock or the weft write lock

**Location:** `_lyx/discussion/decision-record.md` D7 step 1 and steps 2–8.

**Issue:** Step 1 waits until the parent lock of the worktree's branch "is free", then nothing takes or holds it.
A child's loom run is a separate process that outlives batten passes, since dobby drives it with continue passes (`internal/dobbyengine/doc.go:36`), so "starts no pass" does not stop it.
Its Finalize takes the parent lock (`internal/loomcli/landingdeps.go:375-382`), commits the parent pair's run records through `parentRecordsCommitter` at the parent's working-tree `AnchorRel` (`internal/loomcli/landingdeps.go:117`, `:465-471`), and merges into the parent's branch.
For a beam no refusal catches the child, because the reed refusal covers only the worktree's own strands.
The commit lands on the moving weft while content is between `A` and `B` and the junctions are gone, and it breaks the recovery tests, which assume no other weft writer.
Found by the interactions, bound-breaker, existing-paths and walkthrough lenses and by the handler.

### [BLOCKING] Not-landed recovery deletes the worktree's `.lyx`

**Location:** `_lyx/discussion/decision-record.md` D7 recovery, Not landed bullet.

**Issue:** `.lyx` is in the wired name set (`internal/fabricengine/junctionnames.go:303`, `structuralNeverCommittedDirs`), so `B/<names>` covers `B/.lyx`.
The bullet first resets and "deletes what `HEAD` does not track under `B/<names>`", which removes the renamed, never-tracked `B/.lyx`, and only then "renames `B/.lyx` back to `A/.lyx` when only `B/.lyx` exists".
A kill after step 4's rename and before its commit loses the worktree's logs, locks and strand state on the re-run.

### [BLOCKING] Step 2 kills passes the refusals and the wait never saw

**Location:** `_lyx/discussion/decision-record.md` D7 prime refusals and steps 1–2.

**Issue:** The refusals run before step 1 writes the record, and dobbies keep starting passes until then.
For the prime, step 1 waits only on runs whose parent worktree is the prime, yet step 2 ends every dobby of the machine, and `planRecords` walks every ppid descendant (`internal/runproc/end.go:316-345`), including passes `spawnPass` started (`internal/beamcli/dobbydeps.go:464-485`).
A beam dobby's pass started between the refusal and the record is killed, possibly mid-Land into that beam.
On any worktree, a pass the dobby spawned just before it saw the record does not yet hold its driver lock (`internal/battencli/drive.go:63`), so the wait reads it free and step 2 kills it.
This breaks the acceptance criterion "waits, with no pass started meanwhile, for a child run's pass in flight".
Merged from bound-breaker F1 and F2, interactions F4 and walkthrough F3.

### [MEDIUM] `lyx adopt` runs unbounded during a move

**Location:** `_lyx/discussion/decision-record.md` D7, the paragraph on the move record and `EnsureLive`'s callers.

**Issue:** The record bounds adopt only through `EnsureLive`.
Adopt's other steps go around that bound (`internal/adoptcli/doc.go` Step 2, Steps 4 and 5, Step 6 and the owner-halt paragraph):
- `Topology.Add` from the prime;
- detached first passes through `battencli.StartFirstPass`, outside dobby;
- `lyx loom resume` in child pairs;
- `lyx gate test --gate finalize` in the prime.
Each writes or sets its working directory under `A` while a prime moves, and D7's list of unguarded starters does not name them, against the rubric's attack-surface rule.

### [MEDIUM] `Add` from a moving worktree takes the wrong anchor

**Location:** `_lyx/discussion/decision-record.md` D5 and D7 step 4.

**Issue:** D5 takes the new pair's anchor from the parent's Location, which `lyxcwd` reads from the working-tree marker.
The fork itself is cut from the weft branch's commit.
A kill after step 4 wrote the marker `B` and before the commit leaves the working tree at `B` and `HEAD` at `A`, so a later `lyx fabric add`, `lyx beam create` or adopt's `Add` wires the pair and builds its origin record at `B` over content and a marker at `A`.
Only `EnsureLive`, the dobby loop and reconcile consult the record.

### [MEDIUM] The integration landed test reads a stale tip

**Location:** `_lyx/discussion/decision-record.md` D7 step 1 and the recovery's landed test.

**Issue:** The record holds "the weft's tip before it", written before step 1's wait.
Weft commits and fast-forwards can still happen meanwhile and before a kill: F1's parent-records commit, a weft pull, whose `PullWeft` fast-forwards before `settleUnmovedWeft` runs when markers agree (`internal/fabricengine/pull.go:310-314`), or `lyx fabric commit`.
Any of them makes `HEAD` differ from the recorded tip, so an unlanded integration reads as landed and the re-run wires at `B` over content at `A`.

### [MEDIUM] The creel-daemon refusal has no way forward

**Location:** `_lyx/discussion/decision-record.md` D7 prime refusals.

**Issue:** The refusal's premise is that the daemons' state sits under the prime's anchor.
Creel's state is `<hub>/_board/.lyx/creel` (`internal/hubgeom/creelgeom.go:24`, `internal/fabricengine/junctionnames.go:167`), and the daemon is started from the hub directory (`internal/creelcli/spawn.go:49`).
It serves "until it is stopped or its binary is replaced" (`internal/creelcli/cli.go:769`), and no `lyx creel` verb stops it.
Any post starts it, so the refusal fires on nearly every prime change with no lyx way forward, against `PATTERN-refusal-way-forward`.

### [MEDIUM] D10 compares a markerless tip as a different marker

**Location:** `_lyx/discussion/decision-record.md` D10 pull and push stop, and D5's list of uses.

**Issue:** D5 applies its markerless value only to D8, an adopt and D10's checkout.
After D8 commits a marker locally, an upstream that gained markerless commits from a machine not yet migrated reads as a different marker, so D8's own spawned push and every later pull and push of that pair stop.
The named integration then applies "D7's move from `U`'s marker" with no marker on `U`.

### [MEDIUM] The checkout way forward re-anchors the branch being left

**Location:** `_lyx/discussion/decision-record.md` D10, checkout refusal.

**Issue:** `lyx fabric anchor --set <target anchor>` commits the new marker and moves content on the current weft branch, the one checkout is leaving.
That branch is shared with every machine that checks it out, so their pulls and pushes stop until each runs an integrating `--set` (D7's last paragraph).
A branch switch thus permanently re-anchors an unrelated branch.

### [MEDIUM] The integration conflict way forward drops later local commits

**Location:** `_lyx/discussion/decision-record.md` D10, the conflict way forward.

**Issue:** With local commits `c1..c3` and a conflict at `c2`, `git -C <weft> reset --keep c2^` also removes `c3` from the branch.
Only `c2`'s changes are saved and written back, so `c3`'s records survive only in the reflog.

### [LOW] The dirtiness claim names no check on a first run

**Location:** `_lyx/discussion/decision-record.md` D7 refusals and "It skips no dirtiness check on the weft".

**Issue:** Only the recovery path runs a dirtiness check.
On a first run, step 4 commits "all of it", sweeping an uncommitted change into the marker commit and leaving an untracked file at `A` outside every junction.

### [LOW] D7's `A` for a markerless worktree disagrees with D5

**Location:** `_lyx/discussion/decision-record.md` D7 opening and D5.

**Issue:** D7 moves from "its checked-out anchor `A`", which for a markerless weft is `lyxcwd`'s fallback, Bolt and then `"."`.
D5 says a markerless branch's anchor, wherever one is written or compared, is the shallowest directory holding a wired name.
On a pair the census missed, once the Bolt marker is gone, the two differ.

### [LOW] The acceptance criterion overclaims what is spared

**Location:** `_lyx/discussion/decision-record.md` acceptance criterion for `lyx fabric anchor <slug> --set B`.

**Issue:** D7 step 2 concedes that a push an ended dobby spawned is ended with it, since `planRecords`' descendant walk applies no spare check (`internal/runproc/end.go:326-343`).
The criterion still says "sparing the pushes and board syncs" without that limit.

## Rejected

- Goal-scope lens: returned no findings; the record meets every goal point of the entry.
- Existing-paths F4's claim that spared pushes break the integration landed test: D10 stops a replaying push when the two tips carry different markers, which every integration case meets; the other writers it names are kept in F6.
- Bound-breaker F1 and F2 and walkthrough F3 kept as one finding (F3) at BLOCKING rather than as separate findings, since one fix covers them.
