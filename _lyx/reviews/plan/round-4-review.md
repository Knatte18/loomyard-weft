---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/12-run-archive-after-reset.md
    summary: On reset's archive-only path the pending-findings guard refuses on HEAD (not at start, or starts with no common ancestor) and names accept-audit, which cannot succeed there and routes back to reset --to start
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/02-batcher-orientation.md
    summary: No Edit bullet in the plan carries the re-sign arrow although cards change the parameters of concrete functions and methods; the gate's caller-uncovered check never sees them
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/20-overview-without-card-index.md
    summary: Card 20 lists only production Go file paths in Edit while its Intent changes the existing splitFraming, and card 22 lists NewPlanDirRotator and loomCLI.wire through bare files
  - id: F4
    severity: LOW
    class: design
    location: _lyx/plan/13-reset-start-archives.md
    summary: Cards 13 and 22 bundle independently reviewable units and add exported symbols called from another package with no card of their own
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/17-recovery-uncommitted-paths.md
    summary: Card 17 re-implements the run-own-state filter over DirtyPaths that card 13 puts in UncommittedPaths instead of calling it
  - id: F6
    severity: LOW
    class: design
    location: _lyx/plan/14-standalone-reset.md
    summary: TestKeepResetRefusal asserts the per-path way forward that the TestResetCmd keep-refusal step in the same card already covers, and costs a budget raise
  - id: F7
    severity: NIT
    class: consistency
    location: _lyx/plan/11-reset-targets.md
    summary: resetCmd Short, the --to flag description and the file header of webstercli/reset.go name only start and pre-fix or only fabric's reset, and no card lists them
---

### [BLOCKING] Archive-only reset with pending findings has no way forward that makes progress

**Location:** `_lyx/plan/12-run-archive-after-reset.md` (the per-condition reset-caller steps) and `_lyx/plan/13-reset-start-archives.md` ("On this path the guard can also refuse on a differing suspect path, on HEAD not at the start, and on starts with no single oldest commit").

**Issue:**
Card 13 makes two of D3's archive-only causes reachable together with the pending-findings guard: the resolved start is not an ancestor of HEAD, and the recorded starts share no common ancestor.
In both, `freshPendingDrop` runs with pending findings or an uncheckable batch and refuses on HEAD: `head != base` for the first, `headBeforeEveryStart` for the second.
For the second cause the refusal is certain, because a HEAD that is an ancestor of every start would be a common ancestor of them.
Card 12 gives the reset caller `lyx webster accept-audit` (or `accept-audit --batch NN`) as the step.
That step cannot succeed in either state:

- `AcceptPendingAudit` calls `reconcileHead` on the last batch head, which walks HEAD's first-parent chain and accepts only the head itself or a clean parent merge above it. A last batch head descends from the start, so if the start is not an ancestor of HEAD, the last head is not on HEAD's chain either.
  Its refusal names `lyx webster reset --to last-batch-head` (card 15), and card 11 makes that target refuse "not an ancestor of HEAD" with `lyx webster reset --to start` as the way forward.
- With no last batch head recorded, `checkSuspectPaths` runs against an empty base and the findings read as uncheckable, which `AcceptPendingAudit` refuses with the `reset --to start` route (card 16 rewrites it to the plain run).
- `accept-audit --batch NN` refuses a start that is not an ancestor of HEAD with `lyx webster reset --to start` (card 15).

So the loop is `reset --to start` → guard → `accept-audit` → `reset --to last-batch-head` or `reset --to start`, for the rewritten-branch case D3 builds the archive-only path for.
The reset caller is sent back to the refused verb, which is the dead end this task exists to remove, and it breaks D3's "none keeps a refusal" for these causes.
Pending findings are the only trigger, so the state is narrow but reachable: the run-exit audit leaves findings, then the branch is rewritten.

**Fix:**
Decide what the archive-only path does with pending findings and say it in cards 12 and 13.
One option that matches D3's bound ("a branch rewritten under the run, and any uncommitted changes, which the next run starts over"): on the archive-only path the guard skips its two HEAD-relative refusals (`head != base`, `headBeforeEveryStart`), keeps the path-based and missing-commit ones, and returns a drop warning per pending finding, as `freshPendingDrop` already does for unverifiable paths.
The alternative is a named step that does clear the state, with a test that follows it to a successful archive.
Either way the refusal-spec row for `reset --to start: pending findings kept` must list only reachable ways forward, and `TestArchiveRunAfterReset` needs a row that drives the rewritten-branch case to its way forward.

### [MEDIUM] No Edit bullet carries the re-sign arrow

**Location:** `_lyx/plan/02-batcher-orientation.md`, `05-loom-preflight-batcher.md`, `08-computed-merriam-base.md`, `11-reset-targets.md`, `12-run-archive-after-reset.md`, `15-reset-sites-repoint.md`, `16-retire-fresh-after-reset.md`, `17-recovery-uncommitted-paths.md`, `22-plan-write-archives-webster.md`.

**Issue:**
The plan writer's stencil and the spec require `` `<glyph>` -> `<new declaration head>` `` on an `Edit` member glyph whose parameters, results or receiver change, and the card then lists every caller.
No bullet in the plan uses the arrow, yet Intents change these concrete signatures:

- card 2: `costBatcher.Batch` and `identityBatcher.Batch` gain `base StartBase`; `PeakContext` gains the same parameter; `startContext` takes the base (`Batcher.Batch` is an interface method and takes none);
- card 5: `NewLoomPreflight` gains the config base directory;
- card 8: `formBatches`, `batchCards` and `ExecutionBatches` take the `StartBase`;
- card 11: `PlanReset` takes the batch number;
- card 12: `freshPendingDrop` and `headBeforeEveryStart` take the caller's re-run steps;
- card 15: `reconcileReportHead` takes the batch number;
- card 16: `fingerprintMismatchWayForward` takes the re-entry step;
- card 17: `RenderRecoveryPrompt` gains the rendered block;
- card 22: `NewPlanDirRotator` gains the `archiveWebster` seam, and is not listed as a member at all.

Without the arrow the gate's `caller-uncovered` check never runs for these members, so a missed caller surfaces only in the post-merge build.
The variable-to-function changes (`reportHeadRefusal`, `freshRestartSteps`, `planBaselineWayForward`) are not re-signs of a function and are not part of this finding.

**Fix:**
Write each of the above as `` `internal/<pkg>#<member>` -> `<new declaration head>` `` and confirm every caller's file or member glyph sits on the same card (the cards already list most test callers); `caller-uncovered` will name any that are not.
Leave `Batcher.Batch` without an arrow and state its change in the Intent, as card 2 does.
This is one gap in the method, so the fixer sweeps all signature changes in the plan rather than only the members named here.

### [LOW] Card 20 lists only production file paths while changing an existing symbol

**Location:** `_lyx/plan/20-overview-without-card-index.md`; `_lyx/plan/22-plan-write-archives-webster.md` for `NewPlanDirRotator`.

**Issue:**
The stencil reserves bare file paths for a test file, a non-Go file or a whole file the card creates or deletes, and asks for the member glyphs a card changes.
Card 20's non-test Go targets are `internal/planparser/parse.go` and `doc.go`, both bare, while the Intent moves `splitFraming`'s index bounds into a shared helper; that is the rubric's member-glyph finding.
Card 22 reaches `NewPlanDirRotator` (re-signed, see F2) and `loomCLI.wire` only through the bare files `planwrite.go` and `loomcli/wiring.go`.

Cards 6, 16 and 21 also list a bare production file (`wiring.go`, `fingerprint.go`) beside member glyphs.
I do not flag those: earlier rounds put the files there on purpose, because `containment-file-overlap` compares one card's member against another card's file target, and `handle-unreferenced` forbids a `plan:` handle for a new symbol nobody else references.

**Fix:**
Add `internal/planparser#splitFraming` to card 20, keeping `parse.go` for the new shared helper (a file and a member of it on the same card do not overlap).
In card 22 add `internal/loomshed#NewPlanDirRotator` (with the arrow, F2) and `internal/loomcli#loomCLI.wire` as members; no other card lists `planwrite.go` or `loomcli/wiring.go`, so `containment-file-overlap` stays clear.

### [LOW] Cards 13 and 22 bundle units and add cross-package exports without a card

**Location:** `_lyx/plan/13-reset-start-archives.md`, `_lyx/plan/22-plan-write-archives-webster.md`.

**Issue:**
Granularity puts a symbol of a module's public surface, or one reused across packages, in its own card.

- Card 13 holds three reviewable units: the pair reset ending in the archive, the archive-only path with `ResetPlan.ArchiveOnly`, and `UncommittedPaths`.
  `UncommittedPaths` is exported from `websterengine`, called from `webstercli` by cards 13 and 14, reused by `AcceptBatchFabricReference`'s dirt check, and (see F5) wanted by card 17; it has no card of its own.
- Card 22 holds the rotation seam in `loomshed` and `shedrecipe`, and a second unit in `loomcli`: `CommitPlan`'s widened pathspec, `planCommitPathspec` and the exported `ArchivedPlanWebsterDirs`, which `loomcli` calls across packages.
  Neither new function has a `Create` bullet, because the `handle-unreferenced` check rejects a `plan:` handle that no other card references; a split gives `ArchivedPlanWebsterDirs` that second card.

**Fix:**
Give `UncommittedPaths` a card before 13, with the accept-audit refactor and its existing `plan:` `Create` bullet, and let cards 13, 14 and 17 `Use` it.
Split card 22's `CommitPlan` pathspec work (`ArchivedPlanWebsterDirs`, `planCommitPathspec`, `TestPlanCommitPathspec`) into a following card.
Declare `ArchivedPlanWebsterDirs` as a `plan:` handle in the rotation card and `Use` it in the new `loomcli` card; `planCommitPathspec` is referenced by nothing else and stays in the Intent with its file as an `Edit` target.
Renumber the prose cross-references ("cards 13, 14", "card 16 runs after cards 13 and 15", and similar) and the Shared Decisions' `Applies to` lines with the new indices.

### [LOW] Card 17 re-implements UncommittedPaths

**Location:** `_lyx/plan/17-recovery-uncommitted-paths.md`.

**Issue:**
Card 17 lists the worktree's uncommitted paths with `Git.DirtyPaths`, "excluding the run's own state as accept-audit's dirt check does".
Card 13 creates `UncommittedPaths(geom Geometry)` for exactly that list, and `recoverSpawn` has `deps.Geom`.
Two copies of the run-own-state filter can drift apart, and card 17's `Uses` does not name the function.

**Fix:**
Have `recoverSpawn` call `UncommittedPaths(deps.Geom)`, add it to card 17's `Uses`, and classify its result by the transcript writes.

### [LOW] TestKeepResetRefusal duplicates the TestResetCmd keep-refusal step

**Location:** `_lyx/plan/14-standalone-reset.md`.

**Issue:**
The card adds `TestKeepResetRefusal` (budget raised by one) for the per-path way forward, a run-written path against a foreign path, and also adds a `TestResetCmd` step asserting "the per-path way forward reaches the envelope".
`KeepResetRefusal` is only called from `resetCmd`, so the existing `TestResetCmd` step over a real standalone repository already covers its behavior at the public surface; both path classes fit in that step with two dirty files.

**Fix:**
Fold the run-written and foreign rows into the `TestResetCmd` keep-refusal step, drop `TestKeepResetRefusal` and the `internal/websterengine` budget raise it needs.
If the function is meant to stay a tested exported surface, state in the card why the CLI step cannot hold both classes.

### [NIT] resetCmd's Short, flag description and file header go stale

**Location:** `_lyx/plan/11-reset-targets.md` (also cards 13 and 14).

**Issue:**
`internal/webstercli/reset.go` carries three sentences these cards falsify without listing them.
`Short` reads "move the task branch back to the run's start commit or the verify gate's pre-fix head".
The `--to` flag description reads "start or pre-fix (required)".
The file header says the verb "performs it through fabricengine's pair-checkout reset" and clears the pre-fix head, while card 13 adds the archive and card 14 makes standalone call `gitrepo`.
Card 11 names the validation message, the usage line and the long help only.

**Fix:**
Add these three to the cards' Intents (11 for `Short` and the flag description, 13 and 14 for the header).

## Focus verdicts

The directive's seven entries, each settled:

1. Test budget: every card that adds a top-level test function raises its package row in the same card, and the remaining test changes are rows or steps in existing tests, so the budget guard holds. The one redundancy is F6.
2. Pinned method lists: only card 10 adds a `*Repo` method; it lists `cmd/lyx/gitrepoboundary_test.go` and states the `ResetKeep` entry. No other card adds or removes a gitrepo method. Not a defect.
3. New required seams and fixtures: card 22 makes `PlanWrite` require `Rework.ArchiveWebster` and lists `fixture_test.go`'s `newTestEnv`; the conformance row and the gate entries build from `newTestEnv` or from `reworkTestEnv`, which replaces `Rework` wholesale and already fills the seam, and `envkit.FullEnv` and the batten seam list already account for it. Card 5's `AnchorPath` is a path field that `newTestEnv` sets. Not a defect.
4. Pending-findings guard: F1.
5. Changed-text sites: the `runCmd` mismatch sentence is card 16's, and the `resetCmd` standalone and "changes no other webster state" sentences are cards 14's and 13's. The remaining stale reset texts are F7.
6. Card sizing: F4.
7. Uncommitted-paths reader: `UncommittedPaths(geom Geometry)` in `websterengine` is a named, exported seam that reads `geom.git()` in-package, so `Geometry.git` staying unexported is no problem. Not a defect; F4 and F5 concern its card placement and reuse.

Prior-round check, done after this review was saved: none of the earlier fixes regressed (budget rows, the pinned run-bound list, the `newTestEnv` fixture, the three-card split, `UncommittedPaths`), and no earlier item was disputed or deferred.
F1 originates in round 3's F4 fix, which chose `accept-audit` as the reset caller's step for the HEAD-based guard refusals; that step holds for a HEAD on the run's own history but not for the archive-only causes.
Round 2's gate notes (`handle-unreferenced`, `containment-file-overlap`) shaped how F3 and F4 are worded.
