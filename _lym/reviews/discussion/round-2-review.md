---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:52
    summary: The pair pushes D1 keeps detached still send the code side before the records side with no takeover read, against the declared PATTERN-pair-remote text and D1's own order claim
    fix: In D1, replace the bullet that keeps the anchor change's push detached with a rule that every pair push left detached (the anchor change's SpawnStatusPush of the pair, fabric's commit spawn, lym fabric sync and add, and the deferred pair push a lift pays) sends the records side first and the code side only once it landed, with D2's takeover read on a rejection, by moving that order into the shared coalescing step (coalescePushSides) that both the synchronous and the detached pushes run; or route those pair pushes through PushPairAnchored. Add an acceptance criterion that an anchor change's push in a pair taken over by another machine leaves origin's code branch unchanged.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:177
    summary: pauserecord's existing readers treat a bolt-hold entry as an operator pause, so CoveredError names lym tender resume <slug> as the way forward, a verb D5 makes refuse for that slug
    fix: Add to D5 a bullet that a slug entry under pauserecord.BoltHoldCaller (new) is read as a hold by pauserecord's own readers, stating that pauserecord.CoveredError words it as held by an adopt on the hold's machine, with D5's way forward (wait for the hold to clear, or ask the operator when that machine has stopped), and that Coverage.Held reads it so the dobby notice, batten overview and tender report show it as a hold; add an acceptance criterion that a loom or shuttle refusal of a run under a bolt-hold entry names the hold's machine and no lym tender resume.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:133
    summary: D3's comparisons match a recorded name against this machine's name and registration is lazy, so an unregistered machine, or one with a copied machine.json, reads a registered machine's owner as its own
    fix: In D3, state that a comparison resolves the recorded name to a registry entry and reads it as this machine only when that entry's id is this machine's id; that a machine with no registry entry reads no registered machine's current or former name as its own; that every machine registers at its first catch-up or dobby start after the upgrade, not only at an owner write; and that a registration whose id already has an entry under another current name that machine did not rename from is refused as a copied machine file, its way forward a fresh id through lym machine name. Add the copied-file case to D3's bound and an acceptance criterion for an unregistered machine sharing a registered machine's name.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:124
    summary: D3 gives the machine id no lifetime - lym machine name rewrites machine.json without it, a machine with no user config dir has no file to hold it, and a machine that lost it under its own hold is refused its own adopt
    fix: In D3, state that every writer of machine.json keeps the id (the File type carries it and lym machine name's rewrite copies it); state what a machine whose machinename.FilePath resolves no directory does at registration (refused, naming the variable to set); and extend D3's lost-file bound to say that a hold its old entry wrote reads as another machine's, so the operator moves it with lym adopt --take-hold.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:138
    summary: The bound of lym machine name --forget-former understates its reach - dropping a former name another machine bears hands that machine every owner, cancel intent and takeover entry still written under it
    fix: In D3, make --forget-former refuse while any owner on the fetched board, or any takeover entry or cancel intent of a run this machine owns, still names the forgotten name, naming those entries and the way forward lym adopt of them first; and restate the bound as what it can let through once that refusal passes.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:79
    summary: D2's takeover read compares the entry's machine name inside fabricengine, which cannot read the registry, so after lym machine name a machine reads its own earlier takeover entry as another machine's and hands its run over to itself
    fix: In D2, state that the caller of the pair push hands fabricengine the set of names that read as this machine (current and former, from machinename.Names and the registry), and that the takeover read compares the entry's machine against that set; add an acceptance criterion that a rename after an adopt hands no run over on this machine.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:181
    summary: D5's claim that an owed push lands before an adopter's takeover reads origin is held by nothing - no ordering, a bolt-hold entry written before the push turned owed blocks its retries, and D4 step 4 archives owed commits in the window between the adopter's owner write and its takeover
    fix: In D5, drop the ordering claim and state that the hold does not hold an owed push; add that a machine drops a run's bolt-hold entry once that run's push turns owed; in D4 step 4, state that a run with owed pushes and no takeover entry on origin first pushes them, records first, before it archives; and state in D2's bound that an owed push landing after the takeover is archived and its step runs again on the adopter.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:226
    summary: A plain lym adopt re-run on the machine that already owns the subjects runs step 2's archive-and-reset over its own owed transitions and the live fix task's branches, before step 4's catch-up could push them
    fix: In D6 step 2, state that a run whose owner before step 1 was this machine is no moved run - step 2 skips it and step 4's catch-up pushes its owed commits; and add an acceptance criterion that a re-run under this machine's own adopt-fix hold leaves the fix task's branches and processes untouched.
  - id: F9
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:403
    summary: The Open risks line on the stop's missing per-run intents stands although the parent's Added-after-Discussion entry settles it and says to drop it
    fix: Remove the whole bullet under Open risks about the hub-stop-button section (lines 405-407), leaving the section with no entries, and cut the parenthetical (Open risks) from D9's rationale at line 336.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:330
    summary: D9's bound and rationale say D1 leaves every transition on origin, but a transition whose push is owed when the stop runs is held by the stop's own pause and stays local
    fix: In D9's bound, after "nothing pushed is lost", add that a transition whose push is owed stays local and D4's catch-up at the next dobby start pushes it; in D9's rationale, change "leaves every run in a pushed state" to "leaves every run at a transition pushed or owed".
  - id: F11
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:88
    summary: The hand-over mark has no stated root and D4 step 4's hand-over writes none, so a catch-up whose local removal fails leaves a pair no D7 rule removes
    fix: In D2, state the hand-over mark lives under the hub's own .lym, keyed by slug, so a pair removal does not take it; in D4 step 4, state that it writes the same mark once its archive lands, before the local removal.
  - id: F12
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:194
    summary: The upgrade's first read states no order between dropping a legacy adopt entry and pushing the pause intent and the hold, so a crash between them resumes the run unasked
    fix: In D5's upgrade bullet, state the order - the pause intents and the Bolt hold are pushed first, and only then are the adopt and adopt-hold entries dropped, so a cut first read runs again from the entries left.
  - id: F13
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:430
    summary: The criterion that the old owner leaves origin's code branch unchanged after a takeover contradicts D2's hand-over, which pushes an owed code tip the last transition names
    fix: Reword the criterion to - after a takeover, the old owner's next transition and its lym loom commit-records put no code commit on origin's code branch beyond the tip the run's last transition on origin names.
  - id: F14
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:17
    summary: Scope's runfsm exception names only D1's rename and refusal wording, while D1 adds the owed push's last error and D5 the hold's machine to runfsm's inputs and verdict
    fix: Change the Scope exception to - bar D1's rename of one input and its owed push's last error, D5's hold machine, and the wording of their refusals.
  - id: F15
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:236
    summary: D6 says the fix task's landing runs the check again but never names lym adopt check as kept, nor what it reads and writes under the Bolt hold
    fix: In D6's Kept line, add lym adopt check, run by the fix task's on_land row, and state it reads the adopt-fix hold's try counts and on green clears the hold and runs orchcli.BeamUp for each subject, as step 8 does.
  - id: F16
    severity: LOW
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:278
    summary: D7's orphan grace counts from a first-sight notice but the record names no mark that keeps that time across a dobby restart
    fix: In D7's orphan bullet, state that the dobby records each orphan's first-sight time in a machine-local file under the hub's .lym, named in internal/lymdirs, written with the notice and dropped with the teardown.
  - id: F17
    severity: LOW
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:223
    summary: D6 says what a hold for the same subjects and for other subjects does, but not a hold whose subjects overlap the adopt's in part
    fix: In D6 step 1 and the --take-hold paragraph, define same subjects as the same set and state that any other set, an overlap included, refuses naming the hold's subjects.
---

### [BLOCKING] Detached pair pushes keep the code-first order

**Location:** `_lym/discussion/decision-record.md:52`
**Issue:** D1 reorders only `PushPairAnchored` and keeps the anchor change's push detached. That push is `SpawnStatusPush(pair, "")` (`internal/fabricengine/anchorchange.go:557`), whose child runs `coalescePushSides`. That child pushes warp before weft (`internal/fabricengine/coalesce.go:152-164`) and reads no takeover on rejection (`pushSideDetached`). Fabric's commit spawn (`commit.go:152`), `lym fabric sync`/`add` and a lift's deferred pair push reach the same child. Its `runHeld` also takes a code flag a synchronous push left (`pendingflags.go:168-173`). So a code side reaches origin before its records side, and after another machine's takeover. That breaks the declared `PATTERN-pair-remote` text "the code branch goes along after the records side, both before the run acts", D1's "the order holds for every caller", and D1's criterion on flagged code sides. All four lenses found it. Round 1's F1 fix retired the foreign-flag respawn but left this holder, so that fix is incomplete rather than regressed.

### [BLOCKING] bolt-hold entries read as operator pauses by pauserecord's readers

**Location:** `_lym/discussion/decision-record.md:177`
**Issue:** `pauserecord.CoveredError` (`internal/pauserecord/record.go:303-310`) words every slug entry's refusal as "way forward: run `lym tender resume <slug>`". D5 makes that verb refuse for a slug only a `bolt-hold` entry covers. So loom's and shuttle's refusals under a hold name a way forward that refuses, against `PATTERN-refusal-way-forward`. `Record.CoverageOf` sets `Held` only for `AdoptCaller` and `HoldCaller` (`record.go:241-245`), so every display reader shows the hold as an operator pause with no machine named. D5 rewords only the act rule's held refusal.

### [MEDIUM] Registry comparisons never use the machine id

**Location:** `_lym/discussion/decision-record.md:133`
**Issue:** This settles the focus question "can two machines read one owner name as their own": yes. A machine registers only at an owner write or `lym machine name`. D3 says a comparison reads a former name "as that machine" but never that it checks this machine's id. So machine B, unregistered with the hostname-derived name `neve` while A is registered as `neve`, reads A's owners as owned here through `OwnedWorktrees` and `ReadRun` and drives them. The refusal fires only when B registers. Two variants break it the same way. If X was renamed before the upgrade and is still unregistered, Y can register X's former name first. And `machine.json` sits in the per-user config dir (`machinename.FilePath`), so a synced or cloned config gives two machines one id, which D3's "tells two machines apart" does not cover.

### [MEDIUM] The machine id has no stated lifetime

**Location:** `_lym/discussion/decision-record.md:124`
**Issue:** `lym machine name` rewrites the file from a fresh `machinename.File{Name, Names}` (`internal/machinecli/set.go:138`), which drops any other key. A rename then loses the id, and D3's lost-file bound makes the machine a stranger to its own entry. A machine whose `FilePath` returns `ok=false` (`machinename.go:42-51`) has no file to hold an id. A machine that lost its file while its own hold stood reads that hold as another machine's and is refused its own adopt, and D3's bound does not say so.

### [MEDIUM] --forget-former can hand records to another machine

**Location:** `_lym/discussion/decision-record.md:138`
**Issue:** The bound says the flag "changes only this machine's own file and entry". Dropping a former name that another machine bears as its current name makes every owner, cancel intent and takeover entry still written under that name read as that machine. Take an owner the rename sweep missed while offline: B's dobby then takes the subject with no adopt and no takeover entry. Under the rubric's attack-surface rule, the bound has to state this reach or close it.

### [MEDIUM] Takeover read cannot see this machine's former names

**Location:** `_lym/discussion/decision-record.md:79`
**Issue:** The takeover entry names a machine, and the Constraints run the takeover read in `internal/fabricengine`. Under `PATTERN-fabric-boundary`, fabricengine learns nothing of the board, where the registry lives. A machine that adopted as `osl46` and was then renamed `osl-1046` reads its own entry as another machine's on its next rejected records push, finds no hand-over mark, and hands its own run over to itself. D3's "every comparison goes through the registry" gives no route for this one.

### [MEDIUM] D5's owed-push ordering is held by nothing

**Location:** `_lym/discussion/decision-record.md:181`
**Issue:** "its owed push passes the gate and lands before an adopter's takeover reads origin" fails on three paths. First, the other machine learns of the hold only at its next Bolt pull while the adopter runs on, and nothing orders the two. Second, a `bolt-hold` entry written while the push was in its first attempt stays when that push turns owed, so D1's gated retries are held. Third, between D6 step 1's owner push and step 3's takeover push, the old owner's catch-up reads the run as owned elsewhere, and D4 step 4 archives its owed commits without pushing them. Walkthrough's "purpose met" for D5 took only the case where the push was owed before the entry was written. Round 1's F9 fix introduced the bullet; its second path is the case F9 raised, still open for a push that turns owed late.

### [MEDIUM] A plain adopt re-run resets the owner's own runs

**Location:** `_lym/discussion/decision-record.md:226`
**Issue:** This answers the focus question on a plain re-run. Under this machine's own hold, step 1 writes only owner and registry, then step 2 "archives local tips origin lacks … resets them to origin's" for each moved run. On the owner those tips are owed transitions, which D1 holds until they land and which D4's criterion says are never reset. Under `adopt-fix` they also include the live fix task's branches under a running agent. D5's upgrade notice and the `adopt-fix` path both send the operator here. D6 never says whether a run already owned here is "moved".

### [LOW] Open risks keeps a line the parent settled

**Location:** `_lym/discussion/decision-record.md:403`
**Issue:** The entry "Added after Discussion (parent, 2026-11-16): The hub stop writes no per-run pause or cancel intent" settles the question and says "Drop the Open risks line on this". The line still stands, and D9's rationale still points to it with "(Open risks)". The rubric makes this a consistency finding whose fix drops the line.

### [LOW] D9's pushed-state claim misses an owed push

**Location:** `_lym/discussion/decision-record.md:330`
**Issue:** D1 has an owed push's retry wait at the gate. The stop's own hub pause holds it, the idle confirmation passes, and the run is ended with that transition local. D4's catch-up at the next dobby start pushes it, so nothing is lost on this machine, but "D1 leaves every transition on origin" and "leaves every run in a pushed state" overstate it.

### [LOW] Hand-over mark has no root, and the catch-up writes none

**Location:** `_lym/discussion/decision-record.md:88`
**Issue:** This answers the focus question on marks. The mark is "a machine-local file under `.lym`" with no worktree named. A mark under the pair's `.lym` would go with the pair it guards. D4 step 4 hands a run over without writing the mark. So when its local-mode removal fails, the pair keeps its board entry and batten record, is neither orphan nor terminal, and D7's mark-driven removal never takes it.

### [LOW] Upgrade first read has no stated order

**Location:** `_lym/discussion/decision-record.md:194`
**Issue:** The legacy entries are the first read's only "not yet done" mark. A first read cut after dropping an entry and before pushing the pause intent leaves the run with neither, and it resumes unasked, against the bullet's own purpose.

### [LOW] Code-branch criterion contradicts the hand-over

**Location:** `_lym/discussion/decision-record.md:430`
**Issue:** D2's hand-over pushes the code tip the run's last transition on origin names when origin lacks it. That changes origin's code branch, which the criterion "leave origin's code branch unchanged" forbids.

### [LOW] Scope's runfsm exception is too narrow

**Location:** `_lym/discussion/decision-record.md:17`
**Issue:** `runfsm.Inputs` and `Verdict` (`internal/runfsm/act.go:12-31`, `:61-74`) carry neither the owed push's last error, which D1 adds, nor the hold's machine, which D5 adds. The Scope exception names only D1's rename and refusal wording.

### [LOW] lym adopt check is left unstated

**Location:** `_lym/discussion/decision-record.md:236`
**Issue:** The fix task's re-check runs through fronttask's `adopt-check` row, which runs `lym adopt check <subject>`. D6 retires the worker, record and step machinery around it, and its Kept line lists only flags. It is implied that the check stays and what it does on green, but neither is stated where the split can cut on it.

### [LOW] Orphan first-sight time has no home

**Location:** `_lym/discussion/decision-record.md:278`
**Issue:** The grace runs "after that notice", so the dobby needs the first-sight time across restarts. No file, writer or root is named, and a dobby restart could reset or skip the grace.

### [LOW] Partly overlapping subjects are undefined

**Location:** `_lym/discussion/decision-record.md:223`
**Issue:** Step 1 distinguishes a hold "for the same subjects" from one "for other subjects", and `--take-hold` refuses "a hold that names other subjects". A hold over {A, B} met by `lym adopt A` falls between the two.

## Rejected

- Walkthrough's D5 "purpose met": it walked only an owed push that came before the hold read; F7 holds on the paths it did not walk.
- Existing-paths' `lym adopt check` finding at MEDIUM: kept at LOW as F15. D6 already states the check's outcome under `adopt-fix` (green clears, failing files the next task, try counts in the hold), and only the verb's name and its kept status are missing.
- Bound-breaker's D9 finding at MEDIUM: kept at LOW as F10. Walkthrough's D9 case shows D4's catch-up at the next dobby start pushes the owed transition, and D2's bound already covers an adopter that arrives first.
- Interactions' lost-file-under-own-hold case: merged into F4.
- Bound-breaker's D5 ordering wording and interactions' D4-step-4 and entry-before-owed cases: merged into F7.
