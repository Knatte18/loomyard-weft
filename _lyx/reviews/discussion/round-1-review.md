---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md:114 (D7 step 1), :110 (prime refusals), :197 (acceptance criterion)
    summary: runproc.End cannot end the dobby alone; it signals every ppid descendant, so step 1 kills the passes it claims to leave running, for the prime also kills every HubOwner records push, and runs before the refusals that are meant to protect those runs
    fix: Rewrite D7 step 1 against what runproc.End does (internal/runproc/end.go:316-345 walks ppid descendants; internal/beamcli/dobbydeps.go spawnPass starts each pass as dobby's child; internal/fabricengine/spawn.go:128 registers pushes under runproc.HubOwner). Either run the refusals first and refuse while any pass the worktree's dobby spawned is in flight (for the prime, any pass in the hub), then end the dobby, stating that End also ends the dobby's descendants; or name a new runproc seam that ends one registered pid without its descendants, as a Decision with its bound. State for the prime what happens to in-flight HubOwner pushes (refuse while one is live, or name that they are ended and respawned). Drop or rewrite the acceptance criterion at :197 that a pass in flight finishes, so it matches the chosen rule.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md:98-99 (D7), :159 (D10 pull refusal)
    summary: A prime anchor change commits and pushes marker B on the main-weft every machine's prime checks out, so every other machine's next prime pull is refused with a way forward, fabric remove and re-adopt, that refusePrimeSlug makes impossible for the prime
    fix: Add to D7 and D10 a rule for the prime on a multi-machine hub. Either refuse a prime anchor change while the hub has primes on other machines (and say how the verb knows), or give other machines' primes a way to follow, such as the prime pull running D7's steps 2 to 6 for the incoming anchor, with its bound and recovery stated. In either case the D10 pull refusal for the prime must name a way forward that exists (internal/fabricengine/remove.go:949 refusePrimeSlug refuses removing the prime), per PATTERN-refusal-way-forward, and an acceptance criterion must cover the second machine's prime.
  - id: F3
    severity: BLOCKING
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md:126-129 (D7 recovery), :199 (acceptance criterion)
    summary: The recovery cannot determine A or whether a change is in flight from the record; a kill before the commit leaves the last trailer an older or inherited one, a child forked after its parent's change inherits the parent's trailered commit, and a kill after the commit looks the same as a finished change
    fix: Rewrite D7's recovery paragraph so each state after a kill is told apart from durable evidence. Take A for an uncommitted kill from the checked-out marker, not from any trailer. Take A from the Lyx-Anchor-From trailer only when the commit that last changed the marker was made in this worktree's own branch after its fork point (state how that is read, such as the commit not being reachable from the parent's weft branch); otherwise treat the anchor as inherited and clean nothing. Define settled as the marker committed and junctions, scratch link, launchers and workspace entry all present at the marker's anchor, and make a --set equal to an unsettled anchor run steps 4 to 7 rather than a no-op. Make the acceptance criterion at :199 name a kill before the commit, a kill between the commit and step 4, and a --set on a child forked after its parent's change.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md:126 (D7 recovery), :216 (Notes for the plan writer)
    summary: The claim that reconcile does not repair an interrupted change is false for a pair with a present weft; Reconcile runs repairPairWiring, which wires at the marker's anchor, A before the commit into moved content, B after it with launchers and workspace files left behind
    fix: Replace the reconcile sentence in D7's recovery paragraph with what reconcile does in each window (internal/fabricengine/reconcile.go:217-227 and repairPairWiring at :391-424; isRawWarpWorktree at :514 runs only on the missing-weft path), and either refuse reconcile on a pair with an interrupted change (state the evidence it reads, such as a dirty weft with a moved anchor, and the way forward naming lyx fabric anchor) or state that the verb's re-run accepts the state reconcile leaves. Drop the third plan-writer note, which names isRawWarpWorktree as the reason.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md:118-120, :127, :132 (D7 step 3, recovery, dirtiness)
    summary: A re-run after an uncommitted kill finds a dirty weft that the verb's own dirtiness check refuses, cannot tell its own moved content from other uncommitted changes, and has no git trace of the never-tracked A/.lyx to B/.lyx rename
    fix: State in D7 how a re-run detects and undoes its own uncommitted move, naming the evidence it reads (for example a record of the move's A and B written under the weft's lock dir before step 3 and removed after the commit), that the dirtiness check is applied after that undo and only to changes outside the move, and how .lyx is located when A/.lyx, B/.lyx or both exist.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md:114, :124, :128 (D7 steps 1 and 7, recovery)
    summary: The set of dobbies step 1 ended lives only in the killed process, so a re-run's step 7 restarts none and they stay down, every dobby of the machine for the prime
    fix: In D7, make step 7 and the re-run restart every dobby whose worktree this verb covers (the worktree's own, every dobby of the machine for the prime) when its lock reads free, through dobbyengine.EnsureLive, instead of exactly the set step 1 ended; or record the ended set durably before ending any. Name the restart in the recovery paragraph.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md:131 (D7 lock bound)
    summary: The no-lock bound covers only a hand start in the worktree, but lyx orch start, lyx adopt and lyx beam start run anywhere in the hub and call dobbyengine.EnsureLive, which restarts any dobby whose lock is free mid-move
    fix: Extend D7's bound to the automatic starts (internal/orchcli/start.go:271, internal/adoptcli/deps.go:112, internal/beamcli/cli.go:71). Either hold the dobby's lock, or a lock EnsureLive honours, for the whole move so EnsureLive skips that worktree, or state that the operator runs none of those verbs anywhere in the hub during the move and what a dobby started mid-move can do.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:goal-scope
    location: _lyx/discussion/decision-record.md:108, :114 (D7 refusals and step 1)
    summary: Passes of child runs whose parent is this worktree can be in Finalize and land into it, under its parent lock, while steps 2 to 4 remove and rewire its junctions; the entry says no run may be in flight in the worktree while it moves, and D7 neither refuses nor bounds this
    fix: Add a D7 refusal for a held parent lock of this worktree (internal/parentlock) and for any child run of this worktree with a pass in flight, naming the way forward (wait, or lyx tender pause on the child), and add it to the refusal acceptance criterion at :198.
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md:159 (D10 pull refusal), :124 (D7 step 7)
    summary: The incoming tip is undefined when the local weft is ahead; after a re-anchor whose async push has not landed, origin carries A and the pair carries B, so the pull is refused and its way forward drops the unpushed marker commit
    fix: Rewrite the D10 pull rule to refuse only a pull that would change the checked-out marker, comparing the marker at the tip the pull would leave checked out (the fast-forward target, or the replay result in settleUnmovedWeft) with the current one, and say that a local-ahead weft whose upstream carries the old marker is not refused.
  - id: F10
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md:139-141 (D8)
    summary: Removing the Bolt marker waits for every counted branch's fetched ref to carry a marker, but reconcile pushes none of its marker commits, so an idle pair's marker can stay local without end and the Bolt marker with it
    fix: Make D8 spawn the detached weft push for each pair it commits a marker in, as other weft writers do, and state that the census completes on a later reconcile once those pushes and a fetch have landed.
  - id: F11
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md:140-141 (D8 census), :182-186 (Open risks)
    summary: A branch this machine never fetched is invisible to a census that makes no network call, and another machine's local main-weft may lag origin, so the Bolt marker can be removed while a new-binary machine still has a markerless worktree that then resolves to the repo root on a subpath hub
    fix: State in D8 that lyxcwd's read on a markerless weft falls back to Bolt's history value when the Bolt marker has been removed, or that D8 records the removed value somewhere the new binary's D2 read still finds, or else add this window to Open risks with the operator step that closes it (reconcile on every machine before any machine's reconcile removes the marker) and keep the census's unseen-branch limit explicit.
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md:140, :143, :202 (D8 census)
    summary: D8 does not say whether a present but empty committed marker counts as carrying a marker in the census; if it counts, the Bolt marker is removed and that pair resolves to the repo root through D2
    fix: Add to D8 that a branch whose committed marker is empty after trimming counts as unmarked in the census.
  - id: F13
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md:45, :140 (D2 step 2, D8 removal)
    summary: Once D8 removes Bolt's .lyx-anchor, a leftover Bolt .fabric-anchor makes every markerless read return ErrStaleAnchorMarker and clone's stale guard refuse every re-clone of a migrated hub
    fix: Add to D8 that the board commit removing .lyx-anchor also removes a .fabric-anchor beside it, and to D4 that clone's stale-spelling guard runs only on the Bolt fallback path, when main-weft carries no marker.
  - id: F14
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md:69 (D4 fresh-bind test)
    summary: The probe is a single-branch shallow clone of the weft's HEAD, Bolt, so it never sees a main-weft marker; D4's claim that the fresh-bind test counts one needs a probe change the record does not state
    fix: In D4, state that probeWeftBinding (internal/fabricengine/recordsprobe.go) also reads the root of main-weft, by name, and that the bootstrap guard's evidence and message (internal/fabricengine/clone.go:192-200) count a marker on either branch; or drop the main-weft half of the sentence and state that .lyx-warp on Bolt keeps the test right.
  - id: F15
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md:50, :106, :114-115, :192
    summary: D2 and the acceptance criterion say an empty own marker is refused before anything is touched, but D7 step 1 ends dobbies before its refusals, the empty-marker one included
    fix: In D7, run the empty-marker refusal, and every other refusal that does not need the dobby gone, before step 1, so a refusal on an empty marker ends nothing.
  - id: F16
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md:89-90 (D6 re-run)
    summary: D6's re-run rule does not say whether a skip on an equal anchor still finishes the wiring, scratch link and launchers at the new anchor after a kill between the marker commit and the wiring, nor what finishes the beam create steps after a kill before the commit
    fix: State in D6 that a re-run with an equal --anchor skips the move but runs the remaining create steps (wiring, scratch link, launchers, workspace files) idempotently, and that a re-run after a kill before the move commit undoes the uncommitted move as D7's recovery does and then moves again.
---

### [BLOCKING] runproc.End cannot end the dobby alone

**Location:** `_lyx/discussion/decision-record.md:114` (D7 step 1), `:110`, `:197`

**Issue:** `runproc.End` walks every ppid descendant of each record under the owner (`internal/runproc/end.go:316-345`), and `spawnPass` starts each pass as dobby's child (`internal/beamcli/dobbydeps.go`, `proc.Detach` only sets a new group).
Step 1 therefore kills the passes it says it leaves running, and the acceptance criterion that a pass in flight finishes cannot hold.
For the prime, `runproc.HubOwner` also owns the records pushes (`internal/fabricengine/spawn.go:128`), and the prime's "any live batten run" refusal runs after step 1 has ended the runs it guards.
Raised by four lenses; kept once.

### [BLOCKING] A prime anchor change strands every other machine's prime

**Location:** `_lyx/discussion/decision-record.md:98-99`, `:159`

**Issue:** The prime is checked out on every machine and shares `main-weft`.
D7 changes it "on this machine" but commits and pushes marker `B` on the shared branch.
Every other machine's next prime pull is refused by D10, whose way forward, `lyx fabric remove` and re-adopt, `refusePrimeSlug` refuses for the prime (`internal/fabricengine/remove.go:949`, verified).
No decision says how the other primes follow.

### [BLOCKING] The recovery cannot tell its states apart from the record

**Location:** `_lyx/discussion/decision-record.md:126-129`, `:199`

**Issue:** `A` is read from the trailer of the commit that last changed the marker.
After a kill before the commit, that commit is an older or an inherited one, so the re-run moves from the wrong source.
A child forked after its parent's change inherits the parent's trailered commit, contradicting "with no such trailer, as for an anchor inherited through a fork".
"Settled" is undefined, so a kill after the commit, with nothing wired at `B`, looks the same as a finished change and a `--set B` no-ops on it.
This answers the focus question on a kill between the step-3 commit and step 4: the outcome is not determined.

### [MEDIUM] The reconcile premise in the recovery is false

**Location:** `_lyx/discussion/decision-record.md:126`, `:216`

**Issue:** `isRawWarpWorktree` runs only on the missing-weft path (`internal/fabricengine/reconcile.go:514`).
A pair with a present weft goes through `repairPairWiring` (`:391-424`), which wires at whatever anchor the marker names.
Before the commit that wires `A` into moved content, and after it wires `B` and leaves the launchers and workspace files at `A`.

### [MEDIUM] Undoing an uncommitted move is unspecified

**Location:** `_lyx/discussion/decision-record.md:118-120`, `:127`, `:132`

**Issue:** The killed run leaves the weft dirty, which the verb's own dirtiness check refuses.
The undo cannot tell its own moves from other uncommitted changes, and the never-tracked `.lyx` rename leaves no git trace.

### [MEDIUM] Ended dobbies are not restarted after a killed run

**Location:** `_lyx/discussion/decision-record.md:114`, `:124`, `:128`

**Issue:** Step 7 restarts "exactly the dobbies step 1 ended", a set only the killed process knew.
A re-run's step 1 ends none, so its step 7 restarts none.

### [MEDIUM] Automatic dobby starts break the no-lock bound

**Location:** `_lyx/discussion/decision-record.md:131`

**Issue:** `lyx orch start`, `lyx adopt` and `lyx beam start` call `dobbyengine.EnsureLive` from anywhere in the hub (`internal/orchcli/start.go:271`, `internal/adoptcli/deps.go:112`, `internal/beamcli/cli.go:71`, verified).
Any of them restarts the ended dobby mid-move.
The bound covers only a hand start in the worktree.

### [MEDIUM] Child-run landings into the worktree during the move are unbounded

**Location:** `_lyx/discussion/decision-record.md:108`, `:114`

**Issue:** The refusals cover only the worktree's own runs.
A child run's Finalize lands into this worktree under its parent lock and can write through `_lyx` while steps 2 to 4 have removed it.
The board entry says no run may be in flight in the worktree while it moves.

### [MEDIUM] The pull refusal's incoming tip is undefined when the weft is ahead

**Location:** `_lyx/discussion/decision-record.md:159`, `:124`

**Issue:** After a re-anchor whose async push has not landed, origin carries `A` and the pair carries `B`.
Read as origin's tip, the rule refuses every pull, and its way forward discards the unpushed marker commit.

### [MEDIUM] Reconcile's marker commits are never pushed

**Location:** `_lyx/discussion/decision-record.md:139-141`

**Issue:** The census waits on fetched refs, but reconcile spawns no push, so an idle pair's marker commit can stay local without end.
The Bolt marker then stays, against the entry's "no marker stays on Bolt".

### [MEDIUM] The census cannot see unfetched branches or lagging machines

**Location:** `_lyx/discussion/decision-record.md:140-141`, `:182-186`

**Issue:** A census with no network call cannot count a branch this machine never fetched, and another machine's local `main-weft` may lag origin.
The Bolt marker can be removed while a new-binary machine still has a markerless worktree, which then resolves to `"."` on a subpath hub.
The Open risks cover only an old binary and local-only pairs.

### [LOW] An empty committed marker's place in the census is unstated

**Location:** `_lyx/discussion/decision-record.md:140`, `:143`, `:202`

**Issue:** If an empty marker counts as a marker, the Bolt marker is removed and that pair falls back to `"."` through D2.

### [LOW] A leftover stale marker spelling breaks a migrated hub

**Location:** `_lyx/discussion/decision-record.md:45`, `:140`

**Issue:** The stale-spelling guard fires when `.lyx-anchor` is absent from Bolt (`internal/lyxcwd/anchor.go:161-166`, `internal/fabricengine/clone.go:359-369`), which D8 makes the normal state.

### [LOW] The fresh-bind probe cannot see main-weft

**Location:** `_lyx/discussion/decision-record.md:69`

**Issue:** `probeWeftBinding` clones only HEAD, which is Bolt, with `--single-branch` (`internal/fabricengine/recordsprobe.go`), so it cannot count a marker on `main-weft` as D4 claims.

### [LOW] The empty-marker refusal runs after step 1

**Location:** `_lyx/discussion/decision-record.md:50`, `:106`, `:114-115`, `:192`

**Issue:** D2 and the acceptance criterion promise that an empty marker is refused before anything is touched, but D7 ends dobbies first.

### [LOW] D6's re-run skip is underspecified

**Location:** `_lyx/discussion/decision-record.md:89-90`

**Issue:** A skip on an equal anchor does not say whether the wiring still owed at the new anchor is done, nor what finishes the create steps after a kill before the move commit.

## Rejected

- goal-scope finding 2 (census can be put off without limit): merged into F10 and F11, which name the cause and the fix.
- interactions F8 and walkthrough findings 1 and 2: duplicates of F1.
- interactions F4, bound-breaker F6 and walkthrough finding 3: duplicates of F2.
- interactions F2, bound-breaker F4 and walkthrough finding 4: merged into F3.
- existing-paths finding 3 (HubOwner pushes ended): folded into F1.
- walkthrough finding 7: merged into F11.
- Board-fit returned no findings; the handler found no leanness or relocation finding: every Decision paragraph carries a requirement the plan writer needs, and the support log holds the deliberation.

## Focus notes

- Markerless anchor rule (D5): the walkthrough lens and the handler find ties and wired names at two depths cannot arise, since a markerless branch predates per-worktree anchors and holds its content at one anchor; a branch holding none falls back to D2's resolution as stated. Not a defect.
- Package ownership: the interactions lens checked the Constraints against the D7 and D10 placement and found them consistent. Not a defect.
- Legacy Bolt marker and `ErrStaleAnchorMarker`: the disagreement is F12, F13 and F14.
