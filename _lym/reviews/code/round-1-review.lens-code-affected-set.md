```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change, I listed what calls, reads, asserts or documents it, and whether the diff updates it.

**Affected sets, per change**

- **`lym board retire-legacy` deleted:** covered. Its callers, cli tests, help test, the `boardcli` `doc.go` and the set-status help text are all updated. The only places that still name it are measurement and research reports (`docs/benchmarks/test-redundancy.md`, `docs/research/cleanup/unused-features-2026-11-18/report.md`), which the prose rules exempt.
- **`self` run-directory fallback deleted (`shedrun.runSegment`, `LegacyPullRequestRecord`):** covered in `shedrun` and its tests, `pattern/PATTERN-shed-driver.md` and `docs/overview.md:1231`. **Not covered:** the stencil `contracts/stencils/loom/loom-template-discussion.md:168` still says "(and the legacy `_lym/shed/self/`)".
- **Key moves deleted (`configreg.KeyMoves`, `moveRunSettings`) and the runprofile template moving `plan_*` under `tapestry:`:** template, `runprofile/doc.go`, `TestTemplateLadder` and the new `TestFormerRecipeNameRefusal` are covered. **Not covered:** the hub's live `profiles.yaml`, and `docs/overview.md:1524`.
- **Landing, gate and loom key-rename migrations deleted (`Migrate` hooks):** code is covered. **Not covered:** `docs/overview.md:1442-1444` and `1521-1523`.
- **One-time base strip deleted:** `pattern/PATTERN-config-layers.md` is covered. **Not covered:** `docs/overview.md:505`.
- **Legacy `asking` outcome deleted (`shuttleengine` `RunState.AskingOffset` and `reconstruct`):** code, comments and tests are covered. **Not covered:** `docs/overview.md:680`.
- **Darn-to-Darn-Write row rename deleted (retired rows, `RenameProducers`):** **Not covered:** `docs/overview.md:1169`.
- **`verifytree` retired failure kinds deleted:** `doc.go`, the tests and `verify_integration_test.go` are covered.
- **`shedbuild` recipe `retired:` map deleted:** `doc.go`, the parse test (a `retired:` map is now refused as an unknown key) and contracts/recipes are covered.
- **`tendercli.RecordPauseIntent` and the `returnOwed` flag deleted:** `doc.go` and its tests are covered.
- **`OwnerCaller` and `AdoptCaller` pause callers deleted:** the `tendercli` tests and `gate.go` are covered. No other reference remains outside a research report.
- **Batcher `retiredKeyError` way-forward reworded:** `doc.go` and the test assertions are covered.
- **`boardengine.WriteHold` added to the test-only allowlist:** allowed, because `pattern/PATTERN-test-only-code.md` admits a function that "writes a record its package alone writes".
- **Declared PATTERN changes** (config-layers, cwd-resolution, fabric-boundary, shed-driver): each lands in `094-pattern.diff` as the summary declares.
- **Sandbox suite F17 removed:** covered, including its results-template line.
- **`docs/shared-libs/lymcwd.md` Bolt-marker fallback:** covered.
- **Commit trailers:** `467b8093` carries six `Loosened-Check:` trailers, each naming a test and a measured cause. It carries no `Root-Cause:` trailer for the fabric gap it fixes, but the answer key asks for no bug fix, so that gap is no finding under the rubric.

**Findings**

1. **BLOCKING, class design.** Location: `internal/configreg/runsettings.go` (`PrepareRunSettings`; `moveRunSettings` deleted).
   - Summary: the run-settings key move that carried `loom.plan_*` into `tapestry` is gone, but the hub's live `profiles.yaml` still holds `plan_max_bounces` and `plan_hard_max_bounces` inside its `loom:` objects.
   - Consequence: `PrepareRunSettings` → `checkLocalDocument` → `ValidateLocal` now meets a key "not in the loom template" and refuses. That would break `checkProfiles` and every tapestry Seed-Child on this hub. The summary's claim that no hub holds old-layout data does not hold for this file.
   - Evidence:
     - `/home/knatte/Code/loomyard-LYMHUB/_bolt/_lym/config/profiles.yaml:20,33,47,61` (`tapestry: {loom: {... plan_max_bounces: ...}, tapestry: {...}}`)
     - base `internal/configreg/keymove.go:46-48` (the loom-to-tapestry `plan_*` moves, applied to run settings by `moveRunSettings`)
     - HEAD `internal/configengine/layers.go:260-262` (`key %q is not in the %s template`)
   - Caveat: I did not verify whether the profiles 3-way reconcile against `profiles.template-base` rewrites those `loom` objects before a run reads them.
2. **MEDIUM, class consistency.** Location: `docs/overview.md`.
   - Summary: the overview still describes migrations the diff deletes:
     - line 505: the one-time base strip is listed as pending
     - line 680: Attach waits on a legacy `asking` record
     - line 1169: a darn run's `Darn` row is renamed onto `Darn-Write`
     - lines 1442-1444: the `require_pr_to_base`, `merge_fix_attempts` and gate `merge-in` renames
     - lines 1521-1524: the `loom.yaml` `webster_*`/`review`/`fix`/`<segment>_*` renames and the plan-key move into `tapestry.yaml`
   - The summary claims `docs/overview.md` was updated.
   - Evidence: `docs/overview.md:505,680,1169,1442-1444,1521-1524`. `git grep` finds no `merge_fix_attempts` in `internal/`, and `require_pr_to_base` survives only in a test.
3. **LOW, class consistency.** Location: `contracts/stencils/loom/loom-template-discussion.md:168`.
   - Summary: the stencil still names "the legacy `_lym/shed/self/`" as a driver-owned directory, but the fallback that read it is deleted and a stale `self` directory no longer redirects anything.
   - Evidence: `contracts/stencils/loom/loom-template-discussion.md:168`, `internal/shedrun/resolve.go` (`runSegment`, now `return ResolveRunID(l, runID)`).

**Tool calls**

1. `git grep` for the deleted symbol names across the repo. It failed on the pathspec syntax, so call 2 reran it.
2. The same `git grep` with fixed exclusions, plus the runprofile template header. My context had no repo-wide search for leftover references.
3. `git grep` for `merge_fix_attempts`, for reconcile rename/migrate wording, and a read of `pattern/PATTERN-test-only-code.md`. I needed the landing rename's readers and the allowlist rule.
4. `require_pr_to_base` readers, `docs/overview.md:1436-1446` and stencil lines 164-170. My context lacked those doc and stencil texts.
5. The gate `merge-in`/`parent-align` keys and a grep of `docs/overview.md` for migration wording. I needed the rest of the stale doc set.
6. Context reads of `docs/overview.md:500-506`, `1166-1170` and `1518-1524`. I needed to confirm each stale line describes deleted behaviour.
