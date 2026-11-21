---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/configsync/configsync_test.go#TestReconcileAll
    summary: "Tests the redundancy report lists as contained are kept with neither a fold nor a //testtiming:keep reason, which PATTERN-test-speed requires."
    fix: "For each contained candidate in the redundancy report, compare its assertions with those of its named covering tests. The candidates are TestReconcileAll, TestReconcileAll_HubWideModuleHasNoWorktreeResult and TestReconcileHubWideAt_PrunesUnknownKeysOnlyWhenAsked in internal/configsync/configsync_test.go; TestBoardRefusesUnconfiguredLabel, TestPromoteMakesNoteATaskAndIsIdempotent and TestPromotePreservesRecipeAndLabels in internal/boardengine/board_test.go; and TestDraftStatus in internal/boardengine/draft_test.go. Where the covering tests assert the same thing, fold the test into one of them or delete it. Where it pins something they do not, put `//testtiming:keep <reason>` directly above its func Test line, the reason naming that assertion, such as the dry-run bytes and per-module results TestReconcileAll checks, or the unchanged bytes and mtime on a repeated promote."
    origin: assist:assist
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/loomengine/config.go#ConfigSpec
    summary: "With the run-settings key move gone, a loom plan key in a run profile or a board entry's run_settings is refused as not in the loom template, with no pointer to tapestry; this hub's profiles.yaml still holds loom.plan_* in every tapestry block."
    fix: "In loomengine's ConfigSpec, add each key the deleted configreg.KeyMoves moved from loom to tapestry to spec.RetiredKeys, with a spec.RetiredWayForward text such as `set it under the tapestry module`. The keys are plan, plan_timeout_min, implementer, plan_lenses, plan_reviewer_model, plan_fixer_model, plan_reviewer_fan, plan_fixer_fan, plan_judge, plan_seed, plan_max_bounces, plan_hard_max_bounces and plan_auto_accept. Follow the shedOnlyKeys loop there, so configreg.refuseRetiredKeys names the new home. Change the `a loom plan_max_bounces is refused` row of internal/configreg/runsettings_test.go to assert that way forward."
    origin: lens:code-correctness
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/fabricengine/resetsubpathcheck.go#recordedAnchorOnWeft
    summary: "The enumeration of compatibility readers is incomplete: the Bolt weft:main anchor-marker read fallback, drift's stored partial-scope translation and the older-binary staged merge-record arm all remain, and the change description neither deletes them nor names them as kept."
    fix: "In internal/fabricengine/resetsubpathcheck.go, delete recordedAnchorOnWeft's fallback that reads the marker at HEAD of weft:main after `branch --show-current`, so a missing prime marker reports not found. Update clone.go's comment near line 140 that names the Bolt legacy marker. In internal/driftengine, drop the ScopePartial-to-full translation and ScopeWasPartial in config.go, baseline.go and verdict.go, and refuse a stored partial with a way forward naming full. In internal/fabricengine/mergelifecycle.go, delete concludeMergeSides' weft arm for an older binary's `staged` WeftOutcome and the exemption for a record without source SHAs, with their doc.go text. Update tests that pin these paths. Alternatively, list any of them under the change description's kept items with the live data that needs it."
    origin: lens:code-scope
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/fabricengine/clone.go:461
    summary: "The kept stale .fabric-anchor refusal tells the operator to rename the marker in the Bolt worktree, but clone now adopts only the prime weft branch's .lym-anchor, so following the remedy lets the clone fall through to the create path at the default anchor."
    fix: "Reword the refusal's way forward in clone.go: commit `.lym-anchor` holding the old marker's content at the root of the prime's records branch (`RecordsBranchName(warpBranch)`), push it, and remove the board's .fabric-anchor; then retry the clone. Extend TestCloneHub_StaleFabricAnchorHardErrors in internal/fabricengine/clone_adopt_test.go to apply that remedy and assert the retried clone adopts the non-default anchor."
    origin: assist:assist
  - id: F5
    severity: MEDIUM
    class: design
    location: internal/fabricengine/anchormove.go#moveWeftContent
    summary: "The fix in 467b8093, which commits a weft name whose only change is a staged deletion, has no test that fails without it."
    fix: "Add a row to TestAnchorChange in internal/fabricengine/anchorchange_integration_test.go: a weft whose only tracked content under _lym is run records, with their deletion staged before the anchor move. Assert that the move's commit carries the deletion and that `git diff --cached --name-only` is empty afterwards."
    origin: lens:code-tests
  - id: F6
    severity: MEDIUM
    class: design
    location: internal/fabricengine/junction.go:632
    summary: "namesWiredInSiblingWorktrees' rewritten loop is unasserted: negating the same-anchor compare at line 632 or the link check at line 637 survives every test."
    fix: "Add rows to TestExcludeLinesToStrip in internal/fabricengine/unwire_exclude_test.go, or a new integration row, that run the sibling walk. Case one: a sibling links the name at the same anchor, and the exclude line is kept. Case two: a sibling holds the name at another anchor or as a plain directory, and the line is stripped."
    origin: lens:code-tests
  - id: F7
    severity: MEDIUM
    class: consistency
    location: internal/hubreconcile/ensure_integration_test.go#TestEnsure_FailedWalkNamesTheFileAndLeavesTheTreeAsItWas
    summary: "The rollback row lost its assertion that a failed walk leaves no reinterpretation marker behind, though the reinterpretation mechanism stays; no Loosened-Check trailer names the removal."
    fix: "In internal/hubreconcile/reinterpret_integration_test.go, add a serial row that registers the test-only reinterpretation through SetReinterpretationModulesForTest and fails through EnsureFailingAfterHubWideWrite. Assert that the board's config bytes are restored and that no ReinterpretedMarker file and no newly written template base remain. Then run Ensure again and assert the marker is committed once."
    origin: assist:assist
  - id: F8
    severity: MEDIUM
    class: consistency
    location: docs/overview.md:1442
    summary: "Docs, comments and a stencil still describe migrations the diff deletes, while the change description claims docs/overview.md was updated."
    fix: "Rewrite or remove each sentence that describes deleted behaviour. In docs/overview.md: line 505 (the one-time base strip), line 680 (Attach on a legacy asking record), line 1169 (the Darn row renamed onto Darn-Write), lines 1442-1444 (the require_pr_to_base, merge_fix_attempts and gate merge-in renames), lines 1521-1524 (the loom.yaml key renames and the plan-key move into tapestry.yaml) and line 1561 (seats becoming advisor-<n>). In internal/boardcli/runsettings.go: the comment at line 21 and the migrateRunSettings name and doc, which now only validate; rename it to validateRunSettings. In internal/fabriccli/fabric.go:647: the comment saying ReconcileHubWideAt prunes retired keys. In contracts/stencils/loom/loom-template-discussion.md:168: the `(and the legacy _lym/shed/self/)` aside."
    origin: lens:code-affected-set
  - id: F9
    severity: MEDIUM
    class: consistency
    location: _lym/landing/summary.md:45
    summary: "The Merge danger section calls the door two-way, but the diff stops writing board.json's draft_status, which an older binary's migrateDraftStatus reads to decide whether to mark open no-status tasks draft."
    fix: "In _lym/landing/summary.md's Merge danger section, set the door to one-way. Explain that a board saved without draft_status makes an older lym, on another machine or after a revert, mark released open tasks with no status as draft. Name the retired-key load change too: a key older binaries dropped on load is now kept by reconciles."
    origin: assist:assist
  - id: F10
    severity: LOW
    class: design
    location: internal/batcher/config.go:56
    summary: "The retired-key refusal says lym config reconcile --apply or lym fabric reconcile removes the key, but Prune keeps a profile the template does not name whole, so for an operator-named profile neither command removes it."
    fix: "In batcher's retiredKeyError, offer the reconcile commands only when profileName is a profile the template names, and otherwise give only the hand delete of the key from that profile. Update internal/batcher/doc.go, and add a TestActive row with an operator-named profile carrying alone_above that asserts the hand-delete-only way forward."
    origin: assist:assist
  - id: F11
    severity: LOW
    class: design
    location: internal/configsync/configsync_test.go#TestReconcileHubWideAt_SeedsFromHubPrimeOrTemplate
    summary: "Changed lines whose mutants survive have no asserting test: configsync.go:482, configreg.go:174, recordsgit.go:92, fabric.go:742 and circling.go:74."
    fix: "Add one assertion each. configsync.go:482: in TestReconcileHubWideAt_SeedsFromHubPrimeOrTemplate, add a row with the hub models.yaml absent, a prime copy holding only an operator alias and a models.template-base present; assert the template alias is added and reported in Added. configreg.go:174: in the configreg Modules test, assert every shuttleengines.ConfigModules() name is registered. recordsgit.go:92: in TestCommitWeft_SeedsFabricArtifactsOnlyAndIsIdempotent, seed twice and assert the exact exclude file content. fabric.go:742: in TestCommand_ReconcileLeavesAMissingWarpBindingUnrecorded, assert warp_binding_detail is present for the unbound hub, and absent in a bound-hub test. circling.go:74: in TestRecordCirclingDecision_WritesPendingFile, assert the written decision is not marked automatic."
    origin: lens:code-tests
---

The diff deletes the migration and compatibility code the board entry asks for, and `lym board retire-legacy` is gone with no caller, help text, doc or sandbox scenario left naming it.
The declared PATTERN changes land as declared, and the refusal texts that remain name a way forward.
The review blocks on test economy and finds several live readers or texts the deletion left wrong.

### [BLOCKING] Contained tests kept without a fold or a keep reason

**Location:** internal/configsync/configsync_test.go#TestReconcileAll

**Issue:** The redundancy report lists seven of the diff's changed tests as contained. `PATTERN-test-speed` requires each kept test to pin behavior no other test covers, and a kept candidate to carry `//testtiming:keep <reason>`. None of them carries one. The assistant compared their assertions with the covering tests' and found that most pin output the covering tests never check, such as dry-run bytes, exact `Preserved` and `Removed` lists, and an unchanged mtime on a repeated promote. That calls for a keep marker, not deletion, and any test whose assertions duplicate its covering tests is folded instead.

### [MEDIUM] Loom plan keys in run settings are refused with no pointer to tapestry

**Location:** internal/loomengine/config.go#ConfigSpec

**Issue:** At the base, `configreg.moveRunSettings` moved `loom.plan_*` into `tapestry` before validation. With it gone, `PrepareRunSettings` → `checkLocalDocument` → `ValidateLocal` refuses `plan_max_bounces` as `not in the loom template` (`internal/configengine/layers.go:260-262`), and the refusal names no way forward. Every `tapestry` block in this hub's `_bolt/_lym/config/profiles.yaml` (lines 20, 33, 47, 61) still holds those keys under `loom`. The hub-wide reconcile's `FollowTemplate` drops them only where they equal `profiles.template-base`, which they do on this hub. A tuned value, or a board entry's `run_settings`, keeps the key and makes Seed-Child refuse. The decision record keeps cheap refusals that point at the new name, and this key move had none to fall back on.

### [MEDIUM] Compatibility readers left behind

**Location:** internal/fabricengine/resetsubpathcheck.go#recordedAnchorOnWeft

**Issue:** `recordedAnchorOnWeft` still reads the anchor marker at `HEAD` of `weft:main` when the prime branch has none (lines 98-106), and `clone.go:140` describes it as "the Bolt legacy marker". The change description says the Bolt marker read fallback is gone, and the new `PATTERN-cwd-resolution` text says a missing marker means `"."`. `internal/driftengine/config.go:237-242` still reads a stored `partial` scope as full and records `ScopeWasPartial`. `internal/fabricengine/mergelifecycle.go:17` still carries an arm for merge records "an older binary wrote". All three serve data the operator confirmed no hub holds, and none is in the change description's kept list.

### [MEDIUM] The stale-anchor refusal's remedy no longer works

**Location:** internal/fabricengine/clone.go:461

**Issue:** The kept refusal tells the operator to `git mv .fabric-anchor .lym-anchor` in the Bolt worktree and retry. Clone now adopts only `weftPath`'s `.lym-anchor`, read at line 443. After the rename, the stale-marker check passes and the clone takes the create path with the requested anchor. A subpath-anchored hub then silently anchors at `"."`, which is the outcome the refusal exists to prevent.

### [MEDIUM] The staged-deletion fix has no test

**Location:** internal/fabricengine/anchormove.go#moveWeftContent

**Issue:** Commit `467b8093` makes the anchor move commit a name whose only change is a staged deletion. Its test changes are fixture adjustments only. No row stages a deletion of the last tracked records and asserts that the move leaves a clean index.

### [MEDIUM] namesWiredInSiblingWorktrees is unasserted

**Location:** internal/fabricengine/junction.go:632

**Issue:** The mutation report shows negations at lines 632 and 637 surviving. `TestExcludeLinesToStrip` builds `siblingWiring` by hand and never runs the sibling walk, so the same-anchor compare and the link check the diff rewrote are pinned by no test.

### [MEDIUM] A rollback assertion was removed without a trailer

**Location:** internal/hubreconcile/ensure_integration_test.go#TestEnsure_FailedWalkNamesTheFileAndLeavesTheTreeAsItWas

**Issue:** The row "a failure after the hub-wide files, the marker and a template base were written…" lost its check that no `ReinterpretedMarker` remains after the failed walk. Its new form checks only the config files. The reinterpretation mechanism stays by design, yet no remaining test fails the walk after a reinterpretation has run. No `Loosened-Check:` trailer on the range names this removal.

### [MEDIUM] Docs and comments still describe deleted migrations

**Location:** docs/overview.md:1442

**Issue:** `docs/overview.md` still describes deleted behaviour. Lines 1442-1444 describe the `require_pr_to_base`, `merge_fix_attempts` and gate `merge-in` renames. Lines 1521-1523 describe the `loom.yaml` key renames. Line 680 describes Attach on a legacy `asking` record, line 1169 the `Darn` row rename, and line 505 the one-time strip. `internal/boardcli/runsettings.go:21` says run settings are migrated in place, though `PrepareRunSettings` now only validates. `internal/fabriccli/fabric.go:647` says `ReconcileHubWideAt` prunes retired keys. The discussion stencil names "the legacy `_lym/shed/self/`" as a driver-owned directory. The change description claims the overview was updated.

### [MEDIUM] Merge danger called two-way over a board-format change

**Location:** _lym/landing/summary.md:45

**Issue:** The diff removes `draft_status` from the board store (`016-internal-boardengine-part3.diff`), so a board this binary saves carries no marker. An older binary's `migrateDraftStatus` reads the marker's absence as a board that still needs its open no-status tasks marked draft. A hub's other machines may still run an older lym. The rubric files a two-way call over a board format another binary reads.

### [LOW] Retired batcher key: the reconcile way forward does not apply to operator-named profiles

**Location:** internal/batcher/config.go:56

**Issue:** `yamlengine.Prune` keeps a per-name entry the template lacks whole (`internal/yamlengine/layers.go:413-414`). So for an operator-named profile carrying `alone_above`, neither reconcile the new text offers removes the key, and the load refuses again.

### [LOW] Mutation survivors on changed lines

**Location:** internal/configsync/configsync_test.go#TestReconcileHubWideAt_SeedsFromHubPrimeOrTemplate

**Issue:** The mutation report lists survivors on lines the diff changes. Each could be killed by one assertion on the diff's own behaviour: the seed-only `!fileAbsent` guard, the engine modules' registration, exclude-seed idempotence, `warp_binding_detail` and the circling decision's `automatic` flag.

## Rejected

- reed `Anchor.UnmarshalJSON` remap of `below-parent`/`top` (assistant): the change description keeps it on purpose with a stated reason, and it guards per-machine strand state rather than hub data.
- No `Root-Cause:` trailer on `467b8093` (affected-set lens and focus): the board entry asks for no bug fix, so the rubric requires none.
- hubreconcile's per-worktree commit leg left for a follow-up (scope lens): it is general reconcile machinery, not migration code, and the change description records the deferral.
- `boardengine.WriteHold` in the test-only allowlist: `PATTERN-test-only-code` admits a writer of a record its package alone writes.
- The `attach.go:316` survivor: it flips only a log call, which no test reasonably kills.
- The affected-set lens's BLOCKING severity for the loom plan keys: this hub's reconcile drops them where they equal the template base, so the defect bites only tuned or run_settings values. It is kept as F2 at MEDIUM.
