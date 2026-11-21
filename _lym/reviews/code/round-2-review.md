---
verdict: APPROVED
findings:
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/driftengine/baseline.go:126
    summary: "Drift still reads stored old-Drift scope records (the legacy partial scope and the old compare, waiver and acceptance fields) at baseline.go:126, check.go:279, verdict.go:102 and verdict.go:177 oldCheckLines, and doc.go:469 still says a config partial is recorded full with a Warn; _lym/landing/summary.md neither deletes these readers nor names them as kept."
    fix: "Delete the stored legacy-scope readers in internal/driftengine. Remove the ScopePartial arm of scopeText (verdict.go:102), the leaveOut-from-scope.Skip branch in check.go:279 with skippedCases if it loses its last caller, oldCheckLines and compareText in verdict.go with the ScopeRecord fields only they read, and ScopePartial in records.go if nothing else uses it. Drop the old-partial wording from recordScope's comment at baseline.go:126. In internal/driftengine/doc.go, delete line 469 (the partial-to-full Warn sentence, which checkScope now contradicts) and the old-partial and legacy-field text at lines 76-78, 85, 374 and 470. Update the tests that pin these paths, keeping current full and off behaviour and tolerant unknown-key reads. If a live in-flight record does need one of these readers, keep only that reader and name it with the data that needs it under Kept on purpose in _lym/landing/summary.md."
    origin: handler
    carries: F3
  - id: F10
    severity: LOW
    class: design
    location: internal/batcher/config.go:59
    summary: "For a template-named profile in the repo default, retiredKeyError says lym fabric reconcile removes a key the template lacks, but fabric reconcile calls ReconcileHubWideAt with PruneUnknown false (internal/fabriccli/fabric.go:670), so pruneSparse keeps alone_above and the load refuses again."
    fix: "In retiredKeyError's LayerDefault arm in internal/batcher/config.go, replace the lym fabric reconcile alternative with the command that prunes the default layer: run \"lym config reconcile --layer default --apply\" from the hub's prime worktree. That command sets PruneUnknown in internal/configcli/hubwide.go:193. Keep the hand delete first and the operator-named-profile hand-delete-only arm as they are. Update the matching sentences in internal/batcher/doc.go (lines 28-30) and the default-layer row of internal/batcher/config_test.go:266 to the new wording."
    origin: assist:assist
    carries: F10
  - id: F11
    severity: LOW
    class: design
    location: internal/configsync/configsync_test.go#TestReconcileHubWideAt_SeedsFromHubPrimeOrTemplate
    summary: "No row seeds a seed-only hub file from a prime copy while a template base exists, so deleting the !fileAbsent clause of the FollowTemplate call at internal/configsync/configsync.go:482 survives every test."
    fix: "Add a template-base fixture field to the TestReconcileHubWideAt_SeedsFromHubPrimeOrTemplate table in internal/configsync/configsync_test.go. Add a row with the hub's models.yaml absent, a prime models.yaml holding only an operator alias, and a models.template-base present. Assert that a template alias the prime copy lacks is added to the written file and listed in the result's Added, which fails once !fileAbsent is dropped."
    origin: assist:assist
    carries: F11
prior:
  - id: F1
    status: resolved
    answer: Every test the redundancy report listed as contained now carries a //testtiming:keep reason naming what it pins, and the current report lists none as contained.
  - id: F2
    status: resolved
    answer: ConfigSpec adds each moved plan key to RetiredKeys and RetiredWayForward with the tapestry way forward, matching the deleted KeyMoves list. The TestPrepareRunSettings row asserts that way forward.
  - id: F3
    status: open
    answer: The Bolt weft:main fallback, the weft conclude arm and the empty-source exemptions are gone. The fixer's rebuttal holds that the stored partial readers read a different datum, but the board entry orders every compatibility path whose data no hub holds deleted, not only config keys. Config now refuses partial, so no run writes a new partial record, and the summary lists no live data that needs these readers.
  - id: F4
    status: resolved
    answer: The refusal names the .lym-anchor commit on the prime's records branch, and TestCloneHub_StaleFabricAnchorHardErrors commits that marker and asserts the retried clone adopts backend.
  - id: F5
    status: resolved
    answer: TestAdd_AtAnotherAnchorCommitsTheDeletionOfTheOnlyTrackedRunRecords reaches the staged-only deletion and asserts it is committed and nothing stays staged.
  - id: F6
    status: resolved
    answer: TestUnwireJunctions_ExcludeLineFollowsTheSiblingWalk keeps the line for a same-anchor link and strips it for another anchor or a plain directory. Negating junction.go:632 fails both kinds of row, so the mutation survivor comes from the report's list of candidate killing tests, not from a missing test.
  - id: F7
    status: resolved
    answer: TestEnsure_FailedWalkAfterAReinterpretationLeavesNoMarkerAndTheNextWalkCommitsItOnce asserts restored config bytes, no marker or template base left, and exactly one marker commit on the next walk.
  - id: F8
    status: resolved
    answer: Every site F8 named is rewritten. The stale drift doc.go:469 sentence follows from F3's partial-scope change, so carried F3 fixes it.
  - id: F9
    status: resolved
    answer: The Merge danger door is one-way, and a bullet above it explains the draft_status and retired-key consequences for older binaries.
  - id: F10
    status: open
    answer: Operator-named profiles now get the hand delete alone. For a template-named profile in the default layer, the message still offers lym fabric reconcile, which keeps the key.
  - id: F11
    status: open
    answer: "The configreg.go:174 and fabric.go:742 assertions landed. recordsgit.go:92 is already pinned by the byte-identical re-seed check in TestCommitWeft_SeedsFabricArtifactsOnlyAndIsIdempotent. The circling.go:74 constant is the round returned beside ErrNotEscalated, which no caller reads. configsync.go:482 still has no asserting row."
---

### [MEDIUM] Drift still reads stored old-Drift scope records

**Location:** internal/driftengine/baseline.go:126

**Issue:** `checkScope` now refuses a config `partial`, and `ScopeWasPartial` is gone, so no run records a new `partial` scope.
Still, `recordScope`'s comment keeps "an old partial" as a recorded value it keeps.
`check.go:279` applies a `partial` record's `Skip` list to the compare.
`verdict.go:102` renders a `partial` scope, and `oldCheckLines` (`verdict.go:177`) renders the old Drift's compare, waiver and acceptance fields.
`doc.go:469` still says "A config scope read from `partial` is recorded `full` with one Warn naming the key", which the refusal contradicts.
The board entry's decision says all migration code goes, and the summary's "Kept on purpose" list names none of these readers.
The round-1 fixer kept them because they read a stored record rather than the config key.
The board entry's scope is any compatibility path whose data no hub holds, and the operator confirmed no hub holds such data.

### [LOW] The default-layer retired-key way forward names a verb that keeps the key

**Location:** internal/batcher/config.go:59

**Issue:** For a template-named profile in the repo default, the refusal reads `or run "lym fabric reconcile", which removes a key the template lacks`.
`lym fabric reconcile` calls `configsync.ReconcileHubWideAt` with `HubWideOptions{Apply: true}` (`internal/fabriccli/fabric.go:670`).
`PruneUnknown` is false there, so `pruneSparse` keeps `alone_above`, and the next load refuses with the same message.
Before this diff, batcher's `Migrate` hook removed the key on that path.
Only `lym config reconcile --layer default --apply` sets `PruneUnknown` (`internal/configcli/hubwide.go:193`).

### [LOW] No test for a prime-seeded seed-only file when a template base exists

**Location:** internal/configsync/configsync_test.go#TestReconcileHubWideAt_SeedsFromHubPrimeOrTemplate

**Issue:** The prime-seed rows write no template base, so `templateBasePresent && !fileAbsent` and `templateBasePresent` behave the same in every row.
Dropping `!fileAbsent` would treat the prime copy as a present hub file.
A template alias the prime copy lacks would then read as an operator deletion and stay missing.
The mutation report's survivor at `configsync.go:482` shows no test catches it.
The round-1 fixer deferred this item.

## Rejected

- The assistant's carry of F8 to `internal/driftengine/doc.go:469` is merged into F3: the stale sentence follows from F3's partial-scope change, and every site F8 named is fixed.
- The assistant's F11 exclude-content and manual-decision candidates agree with this review: both are already pinned or unreadable, as the F11 prior answer says.

## Notes

- Focus 1: a loom plan key in a run profile or run_settings is refused by `configreg.refuseRetiredKeys` with the tapestry way forward, and the key list matches the deleted `KeyMoves`.
  The mutation survivor at `loomengine/config.go:711` drops the `RetiredKeys` entry, which only changes a local `loom.yaml` refusal from "not in the loom template" to one with the tapestry way forward.
- Focus 1, outside the round's new-finding bar: shuttle's retired `claude_prompt_cache_ttl` and `claude_prompt_cache_ttl_roles` keys are `CategoryNone`.
  So a local `shuttle.yaml` or a run_settings `shuttle` object carrying one passes `ValidateLocal` and has no effect, where the deleted key move used to carry it to `claude`.
  It is not a BLOCKING defect, so this incremental round does not raise it.
- Focus 3: the stale `.fabric-anchor` remedy leads to an adopt. `TestCloneHub_StaleFabricAnchorHardErrors` commits `.lym-anchor` on `main-weft` and asserts the retry adopts `backend`.
  Its comment at `clone_adopt_test.go:377` still says the error "names the marker rename as the remedy".
- Focus 4: the redundancy report lists no contained test, and every test it lists as kept carries its `//testtiming:keep` reason.
  The anchor-move fix, the sibling walk and the hubreconcile rollback each have an asserting test, as the F5, F6 and F7 answers name.
- Focus 5: `docs/overview.md`, `internal/boardcli/runsettings.go`, `internal/fabriccli/fabric.go` and the discussion stencil no longer describe a deleted migration.
  The Merge danger call, one-way with blast radius hub, matches the `draft_status` board-format change.
