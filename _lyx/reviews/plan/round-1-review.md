---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lyx/plan/03-finalize-dirty-reason.md (dirtyParentReason form 1 way forward); _lyx/plan/00-overview.md (third Shared Decisions bullet)
    summary: Card 3 rewrites D2's decided merge-in-progress way forward (commit or set aside with plain git before --abort) into save a copy outside the checkout, and the decision record carries no Added after Discussion entry for the change.
    fix: Keep card 3's wording, which is the only one that progresses, and have the fixer send the correction to the parent (ly:orch) so the decision record gains an Added after Discussion entry under D2 recording that the pre-abort step reads save a copy of any change of their own outside the checkout, because a plain git commit there moves HEAD off the record's start and sideConcludeMayHaveLanded (internal/fabricengine/mergeguards.go) then refuses --abort with merge conclude already landed, and a git stash takes the staged merge with it. Until that entry exists, cite it from the overview's Shared Decisions bullet instead of presenting the departure as the plan's own decision.
  - id: F2
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/plan/03-finalize-dirty-reason.md (Tests, TestFinalize_OverRealHub fourth step)
    summary: The real-hub step asserts the stuck reason holds the parent's code checkout path as a raw substring, while MergeGuardError.Checkout is the lyxcwd-resolved warp path, which passes through filepath.EvalSymlinks; card 1 compares the same field only after EvalSymlinks on both sides.
    fix: In card 3's TestFinalize_OverRealHub fourth-step bullet, state that the expected checkout path is filepath.EvalSymlinks(parentCode) before the substring check, matching card 1's comparison of Checkout.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/03-finalize-dirty-reason.md (dirtyParentReason form 3, NotSynced)
    summary: Card 3 states form 2's real reaching state but not form 3's; a genuine parent divergence is likely stopped earlier by step 1b3's origin reconcile, which classifies a code branch both sides moved as diverged and stops the entry, so form 3 at step 4 is reached only with no reconcile wired or a divergence arising between 1b3 and step 4.
    fix: In card 3, after the form 2 reaching paragraph, add one sentence naming form 3's reaching state (the local upstream ref shows a divergence at step 4's unfetched guard read that step 1b3's reconcile did not stop, or no reconcile is wired) and name the NotSynced row of TestFinalize_StuckReasons as its reaching test.
  - id: F4
    severity: NIT
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/05-finalize-goto-guard.md (Tests, the two parentCodeDirtFromFabric rows)
    summary: The bullet says both rows reach parentCodeDirtFromFabric with no git, but the OpenParent row spawns git through List(l.AnchorPath()) and reaches its error through a failing git worktree list.
    fix: In card 5's bullet, replace with no git by saying the recordedParentBranch row spawns nothing and the OpenParent row reaches its error through a failing git worktree list over a non-repository anchor, run under gitkit.HermeticGitEnv() per PATTERN-test-isolation.
  - id: F5
    severity: NIT
    class: consistency
    origin: lens:affected-set
    location: _lyx/plan/04-halt-note-drops-parent-status.md (Edit list)
    summary: Card 4's Intent gives the parentHeader constant a comment saying it stays for parsing older notes, but parentHeader is not among the card's Edit targets.
    fix: Add `internal/haltnote#parentHeader` to card 4's Edit list, after `internal/haltnote#Parse` (checked with lyx loom validate-plan --overlay, no new finding).
---

### [MEDIUM] Card 3 departs from D2's decided merge-in-progress way forward without a decision-record entry

**Location:** `_lyx/plan/03-finalize-dirty-reason.md`, form 1 of `dirtyParentReason`, and the third Shared Decisions bullet of `_lyx/plan/00-overview.md`.
**Issue:** D2 (`_lyx/discussion/decision-record.md:47`) fixes the wording: the operator "first commits or sets aside with plain git any change of their own in `<checkout>`".
Card 3 replaces that with "saves a copy of any change of their own in <checkout> outside it", and the overview records the change as the plan's own Shared Decision.
The replacement is technically right, and the way-forward fork confirmed it: a commit moves HEAD off the record's `StartSHA`, `sideConcludeMayHaveLanded` (`internal/fabricengine/mergeguards.go:400-412`) then reports the conclude may have landed, and `--abort` refuses, so D2's step cannot progress.
The answer key still says otherwise, and the rubric admits a card that overturns a Decision only with an `Added after Discussion` entry recording it.
The acceptance criterion's required elements (the `--abort` reset warning, the role-guard clause, goto and resume) are all still carried.

### [LOW] Card 3's real-hub assertion compares the checkout path without symlink resolution

**Location:** `_lyx/plan/03-finalize-dirty-reason.md`, Tests, `TestFinalize_OverRealHub` fourth step.
**Issue:** `Checkout` is `f.warpPath`, built through `lyxcwd.ResolveWorktree`, whose `gitWorktreeRoot` runs `filepath.EvalSymlinks` (`internal/lyxcwd/lyxcwd.go:148-158`).
The fixture's `parentCode` comes from the hub fixture's temp root, which may be a symlink or an 8.3 short path.
Card 1 already guards the same field with `EvalSymlinks` on both sides; card 3's substring check does not, so it can fail on a platform whose temp root resolves differently.

### [LOW] Card 3 does not state form 3's reaching state

**Location:** `_lyx/plan/03-finalize-dirty-reason.md`, form 3 (`NotSynced`).
**Issue:** `syncedToUpstreamReason` reads the local `@{u}` with no fetch (`internal/fabricengine/mergeguards.go:183-205`).
Step 1b3's origin reconcile runs first and stops an entry whose code branch both sides moved (`BranchDiverged` in `internal/fabricengine/originreconcile.go:136-157`; `stops()` includes it).
D2 mandates the form, so it stays; but the card explains form 2's narrow reach and names its reaching test, and gives form 3 no such sentence, leaving a reader to believe a common production path reaches it.
The guard-reach fork could not confirm within its budget that the reconcile marks the run's entry stopped for the parent branch, so this stays LOW.

### [NIT] Card 5 says a git-spawning row runs with no git

**Location:** `_lyx/plan/05-finalize-goto-guard.md`, the `parentCodeDirtFromFabric` test bullet.
**Issue:** `OpenParent` calls `List(l.AnchorPath())`, which runs `git worktree list` (`internal/fabricengine/worktreelist.go:117-122`).
The row reaches the error as intended; only the claim is wrong, and a git-spawning row falls under `PATTERN-test-isolation`.

### [NIT] Card 4 edits `parentHeader`'s comment without targeting it

**Location:** `_lyx/plan/04-halt-note-drops-parent-status.md`, Edit list.
**Issue:** The Intent says "the `parentHeader` constant stays for that, its comment saying so"; the constant block in `internal/haltnote/haltnote.go:15-25` holds no such comment today, and the card's Edit list names `Note`, `Render` and `Parse` but not `parentHeader`.

## Focus verdicts

- Card 1 granularity: `has` and `withDirt` are private helpers already bundled into card 1 with the type they serve; the three accessors and `codeDirtyPaths` serve cards 2 and 3 and sit in the card that owns the guard error. Not a defect.
- Card order: `codeDirtyPaths` (card 1) is used by card 2, `Fabric.MidMerge` and `Fabric.CodeDirtyPaths` (card 2) by cards 3 and 5, `DirtyParentRefusal` (card 3) by card 5, `finalizeGuard` (card 5) by card 6. Every use follows its creating card. Not a defect.
- Reachability of refusals in cards 3, 5 and 6: each branch names an input and a test; form 2's narrow reach is stated, form 3's is F3; the `isFinalize` and `StartCheck` error branches are fail-loud wiring guards reached by fakes. The driver's start check is reachable from both entry points, since a hand-over park writes only a marker and leaves the status `blocked` (`internal/loomcli/drivewiring.go:373-383`).
- Attack surface on cards 5 and 6: card 5's Intent and Bound state that a read failure and a present merge record admit, why, and that Finalize's own guard stays the authority; card 6's Bound states the check only reads, runs once and can only hand back. Not a defect.
- Card 3's refusal-spec amendment: the spec file sits on card 3's Edit list beside the code change that needs it, and the item's new text matches D2's wording verbatim. Not a defect.
- Fidelity: D1 through D5, every Constraint and every acceptance criterion are carried, including the 20-path cap, the run-record criterion (the existing `TestMerge_DirtyTargetHalts`) and `Parse`'s tolerance of the old header (card 4). No card edits the task's own board entry. The one departure is F1.
- Test economy and member glyphs: `pairDirtyReason` carries the re-sign arrow, `resumeOpenAttempt` does too, and planned tests extend existing tests except `TestIsFinalizeRow` and `TestFinalizeDirtGuard`, which cover new public or composed behavior no existing test reaches. Not a defect.
- Verify lines: no card carries a `**Verify:**` line, which the spec makes optional. Not a defect.

## Rejected

- guard-reach, an operator or orch `lyx loom step` or `lyx loom run` on a blocked Finalize re-steps it unguarded: D5 scopes the check to a Go driver starting at a blocked stop, and the plan carries that scope faithfully; asking for more would be scope the answer key does not license.
