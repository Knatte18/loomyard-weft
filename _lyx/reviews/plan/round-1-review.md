---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    location: _lyx/plan/02-midmerge-probe.md (ImpactSummary)
    summary: ImpactSummary names foreignMergeStatePresent's callers as the merge verbs' guards, but its six call sites include Fabric.Commit, which is not a merge verb, and it gives no count
  - id: F2
    severity: LOW
    location: _lyx/plan/03-start-refuses-mid-merge.md (ImpactSummary)
    summary: ImpactSummary restates the card's Intent instead of concluding on the blast radius of the edited loomCLI struct, runDriverSpawnAndWait and startCmd
  - id: F3
    severity: NIT
    location: _lyx/plan/02-midmerge-probe.md (Intent, third MidMerge bullet)
    summary: The extracted four-probe reader goes into midmerge.go, splitting the foreign-state probes out of mergestate.go, whose file header says it holds them
  - id: F4
    severity: NIT
    location: _lyx/plan/03-start-refuses-mid-merge.md (Intent, Seam paragraph)
    summary: The rename of TestNewLoomCLI_SetsBothInjectedSeams is left conditional, although a third asserted seam makes Both wrong in every case
---

### [LOW] Card 2's ImpactSummary misnames the caller set

**Location:** `_lyx/plan/02-midmerge-probe.md`, `**ImpactSummary:**`

**Issue:** The summary says `foreignMergeStatePresent`'s callers are "the merge verbs' foreign-state guards".
The function has six call sites in four files:
`Fabric.MergeIn` and `Fabric.Merge` (`merge.go`), `recheckMergePreconditionsUnderLock` (`merge.go`, the under-lock re-check for both), `mergeStateOrForeignErr` (`mergelifecycle.go`, shared by `MergeContinue` and `MergeAbort`), `MergeStageResolved` (`mergestage.go`), and `Fabric.Commit` (`commit.go`).
`Commit` is not a merge verb, so the summary understates the blast radius of the extraction, which is exactly what the reviewer of that card needs to weigh.

**Fix:** Name the count and the sites, including `Fabric.Commit`, and state that every one sees an unchanged answer and error wrapping.

### [LOW] Card 3's ImpactSummary restates the Intent

**Location:** `_lyx/plan/03-start-refuses-mid-merge.md`, `**ImpactSummary:**`

**Issue:** The summary repeats what the card does ("start now refuses a spawn or resume over a mid-merge pair with a new envelope kind") and names the unchanged verbs.
It draws no conclusion over the card's Edit targets:
`runDriverSpawnAndWait` has one production caller (the bootstrap in `startCmd`, `start.go`);
the new `loomCLI` field is set only by `newLoomCLI`, which both `Command()` and `StartAliasCommand()` use, and is left nil by every other `&loomCLI{...}` test literal, none of which reaches `runDriverSpawnAndWait` except through `newTestSpawnAndWaitReceiver`;
and the one out-of-package reader of the start envelope's kind, batten's `childNotParked`, treats the new kind as a hard error.

**Fix:** Replace the summary with a one-line conclusion over those callers and consumers.

### [NIT] The extracted reader belongs beside foreignMergeStatePresent

**Location:** `_lyx/plan/02-midmerge-probe.md`, Intent, the "Read the four git-level probes" bullet

**Issue:** The card puts the new unexported four-probe reader in `midmerge.go` and moves `foreignMergeStatePresent`'s doc comment with it, while `foreignMergeStatePresent` itself stays in `mergestate.go`.
`mergestate.go`'s file header says the file holds "the foreign-state probes the sibling-verb guards ... and the four mutating merge verbs use", so the move splits those probes across two files and leaves that header describing code that is no longer there.
The reader serves the merge verbs' guards and `Commit` as much as it serves `MidMerge`.

**Fix:** Place the reader in `mergestate.go` directly above `foreignMergeStatePresent`, and have `midmerge.go` call it.

### [NIT] The test rename is left to the implementer's judgment

**Location:** `_lyx/plan/03-start-refuses-mid-merge.md`, Intent, the "Seam." paragraph

**Issue:** The card says to rename `TestNewLoomCLI_SetsBothInjectedSeams` "only if its name then misstates what it pins".
Once the test asserts a third field, "Both" is wrong whatever the implementer decides, so the condition always holds and the hedge only adds a decision point.

**Fix:** State the rename outright, for example to `TestNewLoomCLI_SetsInjectedSeams`, with its doc comment updated to match.

## Focus verdicts

- **Card 3 seam wiring:** not a defect.
  Every `runDriverSpawnAndWait` call in `start_driver_test.go` builds its receiver through `newTestSpawnAndWaitReceiver`, directly or through `newResumeBranchReceiver`, so each one gets the `MidMergeNone` fake.
  No other test calls `runDriverSpawnAndWait`, and production receivers come only from `newLoomCLI`.
  The real-`MidMerge` row over the receiver's location fails inside `newPaired`'s `requireDir` on the weft sibling path, which the receiver never creates, so no git is spawned (Test Tier Purity holds).
- **Card 3 refusal placement:** not a defect.
  `mustSpawnDriver` is `!runLockHeld && !driverStrandLive`, so `mustSpawn` and a live strand are mutually exclusive.
  Reading the marker once and probing only on `mustSpawn` or live-plus-marker leaves stale-marker removal, resume, the no-marker running-driver attach and the `driver_not_parked` hand-back refusal unchanged.
  The test rows cover both arms, the parked resume with no line sent, the released bootstrap lock, the stale marker left on disk, the empty `conflicts` list, the probe error, the unopenable pair and the unchanged clean path.
  `output.Err`/`output.ErrFields` set the non-zero exit, as for every existing refusal row.
- **Card 2 probe behaviour:** not a defect, apart from placement (F3).
  The extraction keeps the four calls and their wrapping.
  Resolving geometry only when a side has a conflicted path does not default anything to clean, because the classification never depends on geometry.
  Geometry errors and unmappable paths are errors.
  The weft-side foreign rows use `_lyx/plain-conflict.txt` under anchor `.`, which `unifyConflictPaths` passes through by identity (`_lyx` is a structural wired name), so the stated expectation is the one-repo form.
- **Granularity:** not a defect.
  Card 1 is the smallest change that builds and commits on its own, as the writer stencil defines a card, and it lives in a different package from card 3's targets.
  Card 2 correctly bundles the extraction with the probe it exists to serve.
- **ImpactSummary on cards 2 and 3:** see F1 and F2.
- **Decision-record fidelity:** not a defect.
  Every Decision, Constraint and acceptance criterion is carried: the probe and its three states (card 2), the kind (card 1), the placement and the per-case messages (card 3), and the docs (card 2 `doc.go`, card 3 `Long` and `docs/overview.md`).
  The `doc.go` reword is licensed by the record's Scope, and the test rename follows from the seam card 3 must add.
