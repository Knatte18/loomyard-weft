---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:141 and :220-222 (D8 filter against D13 ready notice)
    summary: D13 sends a beam task's ready notice to the prime orch when the beam's orch is not live, yet D8's stencil filter bars the prime orch from starting any entry with a non-empty parent_beam
    fix: In D13, say the Land row's ready notice names a task to the beam's own orch only when the walk from the task's ParentBeam worktree finds that beam's orch live, and names it in no notice otherwise, the same as the beam-absent case; extend the D13 acceptance criterion with "a ready task of a beam whose orch is stopped or never started is named to the prime's orch not at all". If the prime's orch is meant to see such a notice, instead say in D8 what the prime orch does with a notice for an entry whose parent_beam is a beam, and add that to the orch stencil rule.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:169 (D10 step 4) and :253 (Constraints, PATTERN-fabric-git)
    summary: D10 step 4 removes the pair's weft content with no stated route through fabricengine's destruction chokepoint and no stated treatment of uncommitted weft content
    fix: In D10 step 4, state that the reset runs as an executor in `internal/fabricengine/destroy.go` through the shared check pipeline (containment, ownership, dirtiness, force), and that uncommitted record content in the pair's weft is committed under `fabricengine.ScopedPathspec` before the reset so the weft history keeps it, as `Remove` does before its archive tag. Add a Constraints bullet for PATTERN-fabric-git naming the destruction chokepoint for step 4, and an acceptance-criterion clause to D10 that a promote over an uncommitted weft file leaves the file reachable in the weft's history.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:198-200 (D11 Bound)
    summary: D11's bound names only the conflict session's resolution as what lands unreviewed, but the dropped large-merge reroute also lets through a textually clean merge over files both sides changed
    fix: In the first D11 Bound, replace "what this lets through is the conflict session's resolution of the beam against its parent's newer work" with a sentence naming both cases, the conflict session's resolution and a clean merge of the parent's newer changes to files the beam also changed, and say the same `finalize` gate over the merged tree bounds both.
  - id: F4
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:98-100 (D5)
    summary: D5 states twice that Add refuses no tracked dirt when Parent is named, once for the parent's worktree and again for both worktrees
    fix: In D5, delete the sentence that begins It does not refuse tracked dirt in a named parent's worktree, and fold its clause that the dirty paths are logged at Info into the next paragraph, which already states the rule for both worktrees, so the rule is said once.
---

### [MEDIUM] D13's ready notice reaches an orch D8 bars from acting on the task

**Location:** `_lyx/discussion/decision-record.md` D13 (lines 220-223) and D8 (line 141).
**Issue:** D13 walks from the `ParentBeam` worktree upward to the nearest live orch and falls to the prime "whatever its state when none is".
For a task of a present beam whose orch is stopped or was never started, the ready notice therefore goes to the prime's orch.
D8's orch stencil rule has the prime's orch read and start only entries with an empty `parent_beam`, and this task's `parent_beam` is the beam.
The notice names a task the receiving orch is told not to start, and D13 does not say what the orch does with it.
The beam-absent case was settled the other way, named in no notice, and the stopped-orch case is left in between.

### [MEDIUM] D10 step 4 removes weft content outside the stated destruction route

**Location:** `_lyx/discussion/decision-record.md` D10 step 4 (line 169) and the PATTERN-fabric-git Constraint (line 253).
**Issue:** Step 4 "resets the pair's weft content to what a fresh fork from its parent's weft leaves", which deletes the stopped run's discussion, plan and run records.
`PATTERN-fabric-git` allows a destructive primitive only in `internal/fabricengine/destroy.go`, behind a pipeline whose dirtiness check exists for exactly this question: content in the weft worktree that no commit holds.
`destroy.go` already lists hard resets and moving a pair's branches to origin's tips as primitives behind that pipeline.
The Constraints bullet says only that the git steps of D10 run inside `internal/fabricengine`, which does not place the removal behind the pipeline.
D10's Bound states that nothing of the run survives but its warp commits, and does not state what bounds the loss of weft content that was never committed.
A plan writer reading the record cannot tell whether the removal is a gated executor or a plain commit of deletions.

### [LOW] D11's bound understates what the dropped reroute lets through

**Location:** `_lyx/discussion/decision-record.md` D11 Bound (lines 198-200).
**Issue:** `internal/landingshed/finalize.go` `mergeParent` reroutes a merge as large both when it needed the conflict session and when the parent's changes since the merged tip touch a file the task changed.
D11 drops the reroute for both, since the beam's Finalize runs "a large merge ... like any other", but the bound names only the conflict session's resolution.
A clean textual merge over files both sides changed also lands with no review of the merged result, bounded by the same gate.

### [NIT] D5 says the same rule twice

**Location:** `_lyx/discussion/decision-record.md` D5 (lines 98-100).
**Issue:** The paragraph "It does not refuse tracked dirt in a named parent's worktree" and the following "With `Parent` named, Add refuses no tracked dirt at all, neither the named parent's nor the acting worktree's" state one rule, the second covering the first.
The second was added in round 2 without removing the first.

## Rejected

- Focus 1, the beam landing path: D11 states the exemption.
  The start guard's left-pair and tracked-dirt refusals give way for a project entry whose `beam` is true and whose local pair carries the beam mark, a project with an unreadable origin record keeps the left-pair refusal and names `lyx fabric reconcile` rather than `lyx batten cancel <slug> --remove`, and the D11 acceptance criterion covers a beam from `create` and one from `promote`, also with the prime dirty.
  `internal/battencli/wire.go` `CreateWorktree` skips when `taskWorktreeComplete` finds the pair present and complete, which both beams are, so the landing reaches the skip.
- Focus 2, pair creation under a dirty prime: D5 states that with `Parent` named Add checks neither the named parent's nor the acting worktree's tracked dirt, D7 limits the start guard's dirt refusal to a pair forked from the prime, and the D7 acceptance criterion starts a beam member while the prime is dirty.
  `internal/fabricengine/add.go` `add` refuses on `SourceDirtyPaths` of the acting location unconditionally today, so D5's change is a stated change to that line, not an unnoticed one.
- Focus 3, owner and parent reading from an origin record: D4 states that a found record's parent slug is its `ParentWorktree`, the prime by `ParentBranch` or worktree name, never matched against live worktrees, and its acceptance criterion covers a member whose beam pair is absent here and a prime worktree with another name.
  D13 states that a task whose beam worktree is absent here is named in no ready notice.
  `hubgeom` importing `boardengine` creates no import cycle, since `boardengine` imports neither `hubgeom` nor anything that does.
- Focus 4, the widened guard: D8 states what a session in a beam can reach, and bounds it by the beam test, by the refusal in a beam whose entry is run-held by its landing, and by D11's refusal of an orch start in such a beam.
  D11's bound states what lands unreviewed and what bounds it, apart from the gap in F3.
- Focus 5, board layering: D1 states the `Project` layer for an open project and `Running` for one its landing holds, and that no project counts as a dependency's depth, which is the case `ComputeLayers` in `internal/boardengine/layer.go` would otherwise answer Ready with no kind test.
- The declared PATTERN changes: the `PATTERN-cwd-resolution` text in `## PATTERN changes` matches the declared text, `PATTERN-pair-parent` is added as declared, and D5, D7 and D9 pick a board-derived parent through `ParentBeam` with the prime as the acting location, never the acting worktree.
- The rubric's other bullets: no Decision leaves a fact to be found out later, none edits this task's own board entry, and each new verb, flag and widened guard (`--adopt-local`, the hand-set `beam` flag, D8's guard, D11's start-guard exemption) states what it lets through and what bounds it.
