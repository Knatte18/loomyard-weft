# Card 2 — cmd-lyx-prune

**Edit:**
- `cmd/lyx#`

**Intent:**
Run the prune procedure (Shared Decisions, `prune-procedure`) on `cmd/lyx`'s test files.

Most candidates here are guard tests whose covering test is a whole-tree scan.
Decision 3 governs them: a self-check that proves a guard fires on an offending input (`*_NotVacuous`, `*_DetectionHelper`, `*FiresOnOffendingStaysQuietOnCompliant`, `*ScanSelfCheck`, the `TestFirstBannedToken_*`, `TestFindLongLiteralSleep_*` and `TestIsTierTagged_*` helper checks and their kind) is kept with `//testtiming:keep <reason>` naming the guard it proves non-vacuous, or folded with siblings of the same shape into one table that keeps every offending and compliant row.
The CLI-tree and help candidates (`TestHelpTree_*`, `TestJSONHelp_*`, `TestRunUnknownModule`, `TestExitCode_UnknownModule`, `TestUpdateCommandRemoved`) and the `TestNotifyExitAndSweep_*` and `TestExitSweepBounds_*` families are fold leads.

`TestExitCode_HelpPaths` and `TestRunNoArgs` are this package's `failed` entries (Decision 8).

A guard's allowlist or pinned set that names a file in a core package or under `internal/testkit/` stays as is (Decision 1); the card edits it only when an in-scope deletion requires it.
`tierpurity_test.go` keeps passing over every file this plan moves between tiers, so it is rerun by the plan-level gate after the later cards, not only here.
Guard tests that scan the whole tree must still pass after every later card; a later card that trips one fixes its own files, never the guard.

**ImpactSummary:** Test files of `cmd/lyx` only; every guard keeps its check and its self-check, and production code does not change.

**Commit:** 2: Prune cmd/lyx tests
