---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/configsync/configsync_test.go:1143
    summary: "Retained containment candidates lack the PATTERN-required testtiming:keep explanations."
    fix: "Add precise //testtiming:keep annotations to TestReconcileAll, TestReconcileAll_HubWideModuleHasNoWorktreeResult, and TestReconcileHubWideAt_PrunesUnknownKeysOnlyWhenAsked in internal/configsync/configsync_test.go; TestPromoteMakesNoteATaskAndIsIdempotent, TestPromotePreservesRecipeAndLabels, and TestBoardRefusesUnconfiguredLabel in internal/boardengine/board_test.go; and TestDraftStatus in internal/boardengine/draft_test.go. Name the distinct assertions described below rather than removing their coverage."
    origin: lens:code-tests
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/fabricengine/clone.go:461
    summary: "The stale fabric-anchor refusal prescribes a rename that adoption no longer reads."
    fix: "Replace the board-marker rename remedy with instructions that preserve the old anchor value in .lym-anchor at the prime records root, commit and push that marker on the appropriate prime records branch, and remove the obsolete board marker. Extend TestCloneHub_StaleFabricAnchorHardErrors in clone_adopt_test.go to apply the prescribed remedy and retry with a non-default anchor. Keep the refusal; do not restore automatic migration."
    origin: lens:code-correctness
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/batcher/config.go:56
    summary: "Retired profile-key errors recommend reconcile commands that cannot remove those keys."
    fix: "Remove the ineffective reconcile alternatives from retiredKeyError and direct the operator to delete the named key from the named profile. Update internal/batcher/doc.go and the corresponding batcher/config_test.go and webstercli/verbs_test.go expectations. Do not reintroduce the retired-key migration."
    origin: lens:code-affected-set
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/reedengine/render/types.go:42
    summary: "Old anchor and partial-scope translation paths remain despite the decision to delete obsolete migration readers."
    fix: "Remove Anchor.UnmarshalJSON's below-parent/top translation while keeping the current input refusal pointing to own-window. In driftengine, remove the ScopePartial-to-ScopeFull translation and ScopeWasPartial plumbing in config.go and baseline.go, and the legacy partial-baseline Skip handling in check.go, with their dead constants, branches, tests and documentation. Preserve current own-window and off/full behavior."
    origin: lens:code-scope
  - id: F5
    severity: MEDIUM
    class: design
    location: internal/hubreconcile/reinterpret_integration_test.go:95
    summary: "The retained reinterpretation rollback contract lost its marker-restoration test."
    fix: "Extend the existing serial fake-reinterpretation integration test to fail after the hub-wide rewrite using EnsureFailingAfterHubWideWrite. Assert restoration of the original config bytes and absence of the newly created reinterpretation marker and template-base stamp, then retry successfully and check the committed result."
    origin: lens:code-tests
  - id: F6
    severity: MEDIUM
    class: design
    location: internal/configsync/configsync_test.go:427
    summary: "No seed-from-prime row covers an absent seed-only hub file with an existing template base."
    fix: "Add a template-base fixture field and a row to TestReconcileHubWideAt_SeedsFromHubPrimeOrTemplate. Seed an existing models.template-base containing a template alias, leave hub models.yaml absent, and provide prime models.yaml containing only an operator alias. Check dry-run and apply preserve the operator alias, add the missing template alias and report it as Added. The row must fail if configsync.go drops the !fileAbsent condition passed to FollowTemplate."
    origin: lens:code-tests
  - id: F7
    severity: MEDIUM
    class: consistency
    location: _lym/landing/summary.md:45
    summary: "The two-way merge-door classification ignores the changed board format's effect on an older binary."
    fix: "Classify Merge danger as one-way and explain that boards created after this change omit draft_status, so reverting to the older binary can run its draft migration against released tasks. Keep the hub blast-radius classification and distinguish source rollback from restoring affected board data."
    origin: handler
  - id: F8
    severity: LOW
    class: design
    location: docs/overview.md:1442
    summary: "Live documentation still promises migrations and retired-state handling that this change removes."
    fix: "Update docs/overview.md's legacy asking attachment claim near line 680 and migration descriptions near lines 1442, 1521 and 1559 to describe current behavior. Remove the payload-migration claims from internal/boardcli/doc.go and internal/boardcli/runsettings.go. Correct internal/fabriccli/fabric.go's reconcile help to stop promising retired-key pruning when it does not request PruneUnknown. Preserve unrelated stencil restrictions."
    origin: lens:code-affected-set
---

Reviewed the range `57015a52cd1f436ce96f5137889045e9a2ce5d7d..HEAD` against the board goal and decision record, incorporating all four lens reports. The operator's statement that old hub data is gone authorizes removing obsolete migrations; the findings below concern incomplete removal, current recovery guidance, retained contracts and review requirements.

### [BLOCKING] Explain the retained containment candidates

**Location:** internal/configsync/configsync_test.go:1143

**Issue:** PATTERN-test-speed requires a `//testtiming:keep` explanation when retaining a reported containment candidate. These seven retained tests lack it. Their assertions justify retaining them: `TestReconcileAll` checks dry-run bytes, seed-only preservation and per-module results; the hub-wide test checks absent board/fabric worktree results and unchanged worktree state; the pruning test checks exact Preserved/Removed lists. The board tests respectively check unchanged bytes and modification time on repeated promotion, preserved recipe and ordered labels, rejection without creating a board file plus rejection after vocabulary changes, and draft/null-release behavior through the board facade. Coverage of their functions by other tests does not establish equivalence of these assertions. The round explicitly makes repository PATTERN violations blocking.

### [MEDIUM] Make the anchor refusal's remedy executable

**Location:** internal/fabricengine/clone.go:461

**Issue:** The refusal says to rename the board checkout's `.fabric-anchor` to `.lym-anchor`. Adoption now reads `.lym-anchor` only from `weftPath`, the prime records root; the board-marker fallback was removed. Following the printed remedy therefore suppresses the refusal without supplying the marker adoption reads. A hub whose actual anchor is `backend` can then fall through to the requested default `.`. The retained test checks the error wording and teardown but never follows the advice and retries.

### [MEDIUM] Remove ineffective profile-key recovery commands

**Location:** internal/batcher/config.go:56

**Issue:** Errors for retired `alone_above`, `master_base` and `startup_context` keys offer reconcile commands as a remedy. `profiles` is a PerNameMaps field, and yamlengine reconciliation/pruning carries the named profile mapping intact rather than pruning these nested keys. In addition, `lym fabric reconcile` passes Apply without PruneUnknown. Neither command removes the offending profile key, so the subsequent load still refuses it. Manual deletion is valid; the alternative advertised recovery paths are not. This is the same defect identified by the handler and affected-set lens, consolidated once.

### [MEDIUM] Finish enumerating obsolete readers

**Location:** internal/reedengine/render/types.go:42

**Issue:** `Anchor.UnmarshalJSON` still translates the retired `below-parent` and `top` values to `own-window`. Current input validation already refuses those values with the replacement, so this translation serves stored old data. The landing summary expressly retains it for old stored strands, but the decision record provides no exception to the operator's old-data guarantee. Drift has a second example: config loading translates `ScopePartial` to full and records `ScopeWasPartial`; baseline processing reports that conversion; checking still honors the old partial-baseline Skip list. Current baseline writers use off/full. These are migration readers, not cheap refusals, and leaving both families indicates an incomplete enumeration of the removal goal.

### [MEDIUM] Restore a failure-path test for reinterpretation rollback

**Location:** internal/hubreconcile/reinterpret_integration_test.go:95

**Issue:** The deleted post-failure assertions in `ensure_integration_test.go` included reinterpretation-marker restoration, although reinterpretation remains supported. `walk.go` now selects snapshot paths through `reinterpretationModules`, making correct snapshots of previously absent marker files relevant to this change. The current injected-failure row does not perform a reinterpretation, and the fake-reinterpretation tests exercise success and notifier errors rather than failure after the rewrite. A regression that leaves the completion marker behind can make a retry skip work whose config was rolled back. The retained transaction contract requires config, marker and base restoration together. No Loosened-Check trailer accounts for dropping this assertion.

### [MEDIUM] Pin the missing-file template-base boundary

**Location:** internal/configsync/configsync_test.go:427

**Issue:** The seed-from-prime table has no existing-template-base fixture, while the FollowTemplate table creates the hub file in every row. Neither reaches the combination guarded by `templateBasePresent && !fileAbsent` in configsync.go. For a missing seed-only models file, a prime copy containing only an operator alias must gain the missing template alias even if a base stamp remains. Without `!fileAbsent`, FollowTemplate interprets that absent template alias as an intentional deletion and preserves its absence. This is a missing regression row, not a claim that the current condition is wrong.

### [MEDIUM] Correct the rollback door classification

**Location:** _lym/landing/summary.md:45

**Issue:** The summary calls the change two-way. New boards no longer write `draft_status`. The prior `migrateDraftStatus` interprets its absence as permission to mark each unfinished task with empty status as draft, and records the marker on the next write. Thus a board created under this change and containing released tasks can change meaning after binary rollback. The final-summary specification classifies record/board format changes read by another binary as one-way; reverting source alone does not undo that effect. This does not dispute the approved removal of the migration.

### [LOW] Bring current documentation into agreement with the removal

**Location:** docs/overview.md:1442

**Issue:** The overview still describes migration of retired keys, old loom role names, plan/darn payloads and producer/advisor settings, and still promises legacy asking attachment behavior. Board CLI comments still say PrepareRunSettings migrates payloads, although the current implementation validates them. Fabric reconcile help promises pruning retired entries while its call does not enable PruneUnknown. These descriptions mislead operators and maintainers about current behavior after deletion. This is a documentation correction; static prose tests are unnecessary.

## Rejected

- The affected-set lens also cited the discussion stencil's mention of the legacy `self` directory. That text prohibits writing there; it does not promise a compatibility reader. Removing the restriction is not forced by this cleanup.
- The supporting repair commit's absence of a Root-Cause trailer is not independently a finding: this goal is migration cleanup, and the rubric's mandatory root-cause rule applies to a bug-fix goal. Its changes must still be justified as supporting the cleanup.
- Mutation-survivor labels alone do not establish gaps for fabric seed/sibling behavior, attach PID behavior or circling behavior; the tests identified by the test lens already assert those outcomes. F6 identifies a distinct missing input combination instead.

Validation: the scoped build passed for internal/batcher, internal/configsync, internal/boardengine, internal/fabricengine, internal/shuttleengine, internal/shedbuild, internal/shedrun and internal/verifytree. Tests were not run. The lens reports retain their detailed coverage and evidence lists; this report does not claim runtime validation of the recovery commands.
