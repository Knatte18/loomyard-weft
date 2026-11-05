---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/yamlengine/follow.go:47
    summary: FollowTemplate parses the base even when haveBase is false, so a corrupt base file fails a reconcile that would not read it
    fix: In FollowTemplate, parse the base only when haveBase is true, and otherwise leave baseRoot nil without calling parseMappingRoot; keep wrapping the parse error in ErrInvalidBase when it is parsed. Add a row to the follow_test.go table whose base is "- not\n- a mapping\n" with haveBase false and want no error, the file followed as having no base.
---

### [LOW] A corrupt base fails a reconcile that ignores the base

**Location:** `internal/yamlengine/follow.go:47`, called from `internal/configsync/configsync.go:1046`

**Issue:** `ReconcileHubWideAt` reads the template base for every seed-only module and passes `templateBasePresent && !fileAbsent` as `haveBase`.
When the hub file is absent and a prime copy seeds it, `haveBase` is false, so the base cannot matter, yet `FollowTemplate` parses `base` first and returns `ErrInvalidBase`.
The reconcile then fails naming the base file, though the pass rewrites that base anyway.
The operator is sent to delete a file the pass would have replaced.
The case is narrow (a corrupt base beside an absent seed-only file), so this is LOW.

## Settled focus questions

These are verdicts on the round's directive, not findings.

- **Corrupt or non-mapping base (focus 1).**
  The error names the base file, not the hub file.
  `ReconcileHubWideAt` wraps `ErrInvalidBase` in a `*FileError` whose `Path` is `configengine.TemplateBaseFile`.
  `worktreeErrorFor` drops the module for a template base path, so no `lym config <module>` command is named.
  The way forward is "delete it, and the next reconcile treats <module>.yaml as having no base".
  After the delete, the next walk finds no base, follows without one and rewrites the base, which `WroteTemplateBase` commits.
  So a later reconcile recovers once the base is deleted; until then every start verb fails the same way, which is the stated way forward, not a defect.
  Rows in `TestReconcileHubWideAt_SeedOnlyFileFollowsItsTemplate` and `TestEnsure_FailedWalkNamesTheFileAndLeavesTheTreeAsItWas` reach it, the latter including the repair.
- **Base committed on each path (focus 2).**
  - `lym config reconcile --layer default --apply`: `Result.ChangedFilesRel` lists the base, and the integration test asserts the config commit carries it.
  - `lym fabric reconcile`: `fabriccli/fabric.go` now uses `ChangedFilesRel`, and `TestRunCLI_ReconcileBacksFillsWarpBinding` asserts the base is tracked again after the reconcile.
  - `fabric clone`: the whole-repo board commit carries it, `clone.go` records `file_written` for it, and `TestRunCLI_CloneEndToEnd` asserts both.
- **Re-encoding (focus 3).**
  `follow_test.go` pins a flow mapping past 80 columns staying on one line, and the doc comment on `FollowTemplate` and `docs/shared-libs/yamlengine.md` both state that blank lines between entries are dropped.
  No finding.
- **Branch coverage (focus 4).**
  - Added, updated and kept-the-hub's mail lines and the no-mail rerun: `TestEnsure_CarriesATemplateChangeIntoASeedOnlyConfigFile`.
  - Removed: its Removed assertion.
  - Unreadable base: a `baseIsDir` row in the configsync test.
  - Restore of a base written before a later failure: the "base were written restores…" row of `TestEnsure_FailedWalkNamesTheFileAndLeavesTheTreeAsItWas`.
  No uncovered branch found.
- **Stale wording (focus 5).**
  The remaining "never fills" texts in `configcli/doc.go`, `configcli/hubwide.go` and `configsync.go` are all scoped to a base that is not seed-only.
  No text still says a present seed-only file is not filled.
- **Test doc comments (focus 6).**
  Every test function in `configsync_test.go` that the diff adds or changes sits directly under its own doc comment.
  `TestReconcileAll_Idempotent` has none, but the diff does not touch it.
