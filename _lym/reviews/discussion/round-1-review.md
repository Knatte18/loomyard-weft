---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md#D3
    summary: D3 reads the checklist's window name from the session's reed state, but reed.json records no window name, and the Rationale's window_share_max 0 case is exactly where a read of the strand's group names a window that does not exist; the reader and the derivation are left for Plan.
    fix: "In D3, replace the sentence that reads the window from the session's reed state with a settled source and its pin. Either (a) read reed.json under the worktree's anchor `.lym` directory, find the strand whose role is `loom-status`, and name its window as its `Display.Window` group when `WindowGroups[group].Shared` is true, else the role `loom-status`, pinned to `createOwnWindowPaneLocked` in `internal/reedengine/spawn.go` and `planOwnWindowPlacement` in `internal/reedengine/ownwindow.go`; or (b) ask tmux for the `window_name` of the strand's pane. Name which, and the reader that does it. Rewrite D3's Rationale to drop the claim that it avoids recomputing reed's rule when it uses (a), and keep the window_share_max 0 case only with the derivation that handles it. Extend the D3 acceptance criterion so one case pins the window name at window_share_max 0."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md#D1
    summary: D1 counts an add refused because the loom status role is held as done, but reed's refusal is an untyped fmt.Errorf with no sentinel, so telling it apart is left for Plan; a dead holder is reclaimed rather than refused, and a hidden or paused holder is refused though no checklist shows, while reed's AddStrandUnless already skips an add whose name is held.
    fix: "In D1, replace the role-held refusal sentence with: the re-add goes through reed's `AddStrandUnless` with the loom status role as its unless-name, and a skipped add counts as done and writes no sidecar; orch's strand seam gains that operation. Pin it to `AddStrandUnless` and `strandNameLocked` in `internal/reedengine/strand.go`. Update the Constraints bullet that says reed refuses an add whose role another strand holds to name the skip instead, and change the matching D1 acceptance criterion to: when a strand holding the loom status role already exists, `ensureRunsStrand` adds none and returns no warning."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md#D1
    summary: D1's own add can win against a concurrent lym loom start that read the strands after the runs strand's replace; the bootstrap's AddStrand is then refused on the held role and ensureStatusStrand returns the error, so lym loom start fails, a race D1 opens and the record does not handle.
    fix: "Add to D1 that the loom bootstrap's add of the status strand also counts a strand already holding the loom status role as kept, through the same `AddStrandUnless` route as F2, so neither side of the race fails. Add a D1 acceptance criterion: when the loom status role is taken between the bootstrap's strand read and its add, `lym loom start`'s status-strand step succeeds and adds no second strand."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md#D1
    summary: Two failure paths lose the checklist for the rest of the run, because their warnings name lym beam start and a rerun of it never reaches D1's branch; a ReplaceStrand whose add fails has already removed the status strand, and a failed runs sidecar write returns before the re-add.
    fix: "In D1, state that the re-add runs once the replace succeeded, whatever the runs sidecar write returns, and that the runs sidecar warning is joined with the re-add's own. State that on the loom status branch a failed `ReplaceStrand` returns a warning naming `lym loom start` as well as `lym beam start`, since the status strand may already be gone. Add a D1 acceptance criterion for each path: a failed runs sidecar write still leaves a loom status strand in its own window; a failed replace returns a warning naming `lym loom start`."
  - id: F5
    severity: BLOCKING
    class: decision
    origin: handler
    location: _lym/discussion/decision-record.md#Open risks
    summary: Parent commit 7c23591, on main and on the parent branch plan-writer-explorer-forks but not an ancestor of this worktree's HEAD, fixes the same bug with several overview strands and removes the replace branch D1 builds on; only the run's parent can say whether this bench arm designs against its own base or aligns with that commit.
    fix: The run's parent rules whether the arm ignores commit 7c23591 or aligns with it, recorded with lym loom decision add; on ignore, the writer keeps the Open risks line until the decision is recorded and then drops it; on align, the writer redesigns D1 and D2 against the commit's multi-overview reed.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md#Constraints
    summary: The Constraint that the overview reads files only and takes no lock contradicts D3's reed-state read, since reedengine.LoadState reads through state.ReadJSON, which takes a shared read lock on reed.json.lock.
    fix: "Once F1 settles D3's reader, make the Constraint match it: either D3 reads reed.json without its lock, tolerating a torn read as unreadable, or the Constraint says the overview takes reed's shared read lock for that one read and no other lock. Pin it to `internal/state/state.go`'s `ReadJSON`."
  - id: F7
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md#D1
    summary: D1's premise that the seed, status and sidecar are the session's own worktree is stated without its evidence, and the support log credits bringUpLocation, which only BeamUpWorktree uses.
    fix: "In D1, after the sentence naming the session's own worktree, add its evidence: every `orchcli.BeamUp` caller passes the beam's own location, and `wireCLI` in `internal/orchcli/cli.go` sets the orch's location and its reed geometry from it. In the support log's Turn 3 line on existing-paths 2, replace the credit to `bringUpLocation` with `wireCLI`."
  - id: F8
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md#Acceptance criteria
    summary: D1 makes a failed loom status sidecar write a warning naming lym loom start, but no acceptance criterion covers that path; the failure criterion covers the add alone.
    fix: In the D1 criterion that begins with when re-adding the status strand fails, change it to cover the re-add or its sidecar write failing, each returning a warning naming lym loom start with the runs strand and its sidecar left in place.
  - id: F9
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md#Acceptance criteria
    summary: The D2 criterion clause saying the loom bootstrap and ensureRunsStrand both use the builder is a structural claim, not behavior a test can check.
    fix: Cut the clause the loom bootstrap and ensureRunsStrand both use it from the D2 criterion; the D1 and D2 criteria already check the specs both paths produce.
---

### [BLOCKING] D3 reads a window name reed state does not hold

**Location:** `_lym/discussion/decision-record.md`, D3 and its Rationale.
**Issue:** `Strand` in `internal/reedengine/state.go` persists `Display.Window` and `PaneID`, never a window name.
The name is given to tmux at launch:
the group when the group's record is shared (`spawn.go`, `createOwnWindowPaneLocked`, the shared branch),
the role through `strandLabel` when the strand is placed apart.
At `window_share_max` 0, `planOwnWindowPlacement` records the group as not shared, so the window reads `loom-status` while reed.json still says `pawl`.
The Rationale names this case as the reason to read reed's state, and it is the case a read of the group gets wrong.
Settling it needs the group record or a tmux query, and the record names neither, leaving a fact the design rests on for Plan.
All four lenses reported it.

### [BLOCKING] Parent commit 7c23591 lands a different design for the same bug

**Location:** `_lym/discussion/decision-record.md`, Open risks, first bullet.
**Issue:** `git merge-base --is-ancestor` shows 7c23591 is on `main` and on `plan-writer-explorer-forks`, and not an ancestor of this worktree's HEAD.
Its message says `ensureRunsStrand` no longer replaces the loom status strand and reed admits several overview strands, which removes the premise of D1 and the second Constraint.
The record states it correctly, with the commit named, and leaves the call to the parent.
No `Added after Discussion` entry and no `Decision for wxb-plan-without:` board note answers it.
No Decision depends on its outcome, since D1 to D3 design against this worktree's base.
This finding raises it to the parent.

### [MEDIUM] D1's role-held refusal cannot be told apart from other failures

**Location:** `_lym/discussion/decision-record.md`, D1, and Constraints second bullet.
**Issue:** `strandNameLocked` refuses a held explicit name with a bare `fmt.Errorf`.
The package's sentinels are `ErrNoSession`, `ErrUnknownStrand`, `ErrNewWindowFailed` and `ErrWindowReadBack`, none for this refusal.
`reclaimDeadNameLocked` runs first and removes a dead holder, so a dead strand is not refused.
A hidden holder, or one kept while the pair is paused, is refused and would count as done with no checklist showing.
`AddStrandUnless` skips the add, with no error, while a strand of that name exists, which is the behavior D1 wants.
Orch's strand seam does not expose it today.

### [MEDIUM] D1's add makes a concurrent lym loom start fail

**Location:** `_lym/discussion/decision-record.md`, D1.
**Issue:** `ensureStatusStrand` in `internal/loomcli/sharedbootstrap.go` reads the strands, finds no status strand after the runs strand's replace, and calls `AddStrand`.
If D1's add lands in between, reed refuses the role, and `ensureStatusStrand` returns the error, which fails `lym loom start` at `internal/loomcli/start.go`.
Before D1 that add would have succeeded beside the runs strand.
The second Open risks bullet is about another race, with neither strand present, and does not cover this one.

### [MEDIUM] Two failure paths lose the checklist past any rerun of lym beam start

**Location:** `_lym/discussion/decision-record.md`, D1, and its Bound line.
**Issue:** `ReplaceStrand` removes the old strand and persists the removal even when its add fails.
A failed runs sidecar write returns before any later step.
Both warnings name `lym beam start`.
A rerun of it reaches the add branch or the runs replace branch of `ensureRunsStrand`, never the loom status branch D1 extends, so the checklist stays gone while pawl lives.
Re-adding on every branch would bring back the alternative the support log rejects, restoring a checklist the operator closed.
The fix keeps D1 on its one branch and corrects the order and the way forward instead.

### [LOW] The overview's no-lock Constraint contradicts D3's reed read

**Location:** `_lym/discussion/decision-record.md`, Constraints, last bullet.
**Issue:** `reedengine.LoadState` reads through `state.ReadJSON`, which takes a shared read lock on `reed.json.lock`.
The seed and status reads are lock-free, so the Constraint holds for them and fails only for the reed read F1 settles.

### [LOW] D1's same-worktree premise lacks its evidence

**Location:** `_lym/discussion/decision-record.md`, D1.
**Issue:** The premise holds: every `orchcli.BeamUp` caller, in `loomcli/pawlwiring.go`, `beamcli/cli.go` and `adoptcli/deps.go`, passes the beam's own location, and `BeamUpWorktree` resolves it through `bringUpLocation`.
`wireCLI` sets the orch's location and reed geometry from it.
The record states the premise with no source, and the support log credits `bringUpLocation`, which serves one caller only.

### [LOW] No criterion covers a failed status sidecar write

**Location:** `_lym/discussion/decision-record.md`, Acceptance criteria.
**Issue:** D1 names two failures with the same outcome, the add and its sidecar write.
The criteria test the add's failure alone.

### [NIT] A D2 criterion clause is structural

**Location:** `_lym/discussion/decision-record.md`, Acceptance criteria, first D2 bullet.
**Issue:** "the loom bootstrap and `ensureRunsStrand` both use it" names a call structure, not an outcome a test can check.

## Rejected

- Walkthrough's note that D2's builder needs a location for the seed it reads: the lens did not file it, and both callers hold the worktree's location, so the record need not name it.
- Interactions' claim that the second Open risks bullet is false because D1 widens the race: that bullet describes a different race, with neither strand present, and holds; the new race is kept as F3.
- Leanness: Rationale and Constraints do not restate the Decisions beyond what each needs, and no section holds deliberation that belongs in the support log, so no relocation finding is raised.
