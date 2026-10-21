# Plan round 1 — fixer report

I checked every finding's premise against the code at HEAD, and all nine hold.
`lyx loom validate-plan` returns `ok: true` after every change, the self-check fixes included.

## Changes

- **F1** (card 9), fixed as proposed.
  The `TestLoomResumeReconcilesTheHubConfigBeforeArming` broken-`loom.yaml` step now expects:
  - exit 1;
  - output naming the file, the Warn's `reconcileWayForward` text and the loom config load's own error;
  - no stamp written.

  The "no re-run this command" assertion is gone.
  The load's error appearing after the Warn is what proves the reconcile no longer stops the verb.
  The Bound paragraph now says the way forward for a broken file is the Warn printed above the load's refusal.
- **F2** (card 9), fixed as proposed.
  - I dropped the Edit targets `TestEnsure_FailedWalkNamesTheFileAndLeavesTheTreeAsItWas` and `TestEnsure_HeldLockTimesOutWithoutStamping`.
  - I added the Create target `plan:internal/hubreconcile_test#TestEnsureOrWarn_WarnsWithTheWayForward`.
  - That test is one top-level table-driven test that calls `logcapture.Capture` and never calls `t.Parallel`, modelled on `TestEnsure_UnreachableBoardRemoteSkipsHubWideConfigAndLeavesStampAbsent`.
  - It has rows for a broken hub then the fixed file, and for a held lock.
  - Its comment says the logger is process-global.
- **F3** (card 6), fixed as proposed by factoring the comparison out.
  - New Create target `plan:internal/yamlengine#DropTemplateEqualNames`. It runs after `Prune` for a module with `PerNameMaps`, drops each per-name entry that decodes equal to the template's, and reports them in `Result.Dropped` as `P.<name>`.
  - It shares `sameValue` with `mergePerNameMaps`.
  - New row in `TestReconcileHubWideAt_SeedsFromHubPrimeLegacyOrTemplate`: a present lenses base holding a template-equal lens beside a tuned one, with no `StripBase` set.
  - The ImpactSummary names the `yamlengine` addition.
- **F4** (card 8), fixed as proposed.
  - The `reconcileHubWide` Edit line carries the re-sign arrow with the `hooks walkHooks` parameter.
  - `internal/hubreconcile#walkHub` is added after it.
  - The Intent says `walkHub` passes its hooks to `reconcileHubWide`.
- **F5** (card 3), fixed as proposed.
  - Added the Edit target `internal/battencli#TestLoadOverviewThresholds`.
  - Added a Tests bullet expecting `Soft: 150000` and `Hard: 200000`.
  - Narrowed the "stays as it is" sentence to the `orchThresholds` fixture alone.
  - Replaced the ImpactSummary's "six test members", which the new target made wrong, with "the test members".
- **F6** (card 11), fixed as proposed, through the assertion branch.
  - The Intent says that when the clone's board commit fails, `CloneAndWire` removes a marker it wrote in this call before returning.
  - `TestRunCLI_CloneEndToEnd` gains a second clone run under `t.Setenv` of `GIT_CONFIG_COUNT`, `GIT_CONFIG_KEY_0` and `GIT_CONFIG_VALUE_0`, which point `core.hooksPath` at a `pre-commit` hook that rejects only a commit staging `BaseSparseMarkerRel()`.
  - It expects a non-zero exit naming `pre-commit` and no marker file on the board.
  - The test calls no `t.Parallel`, and its comment names the process environment.
  - I took this branch because the commit can fail; the existing `rejectCommits` rows in `cli_test.go` show a pre-commit hook does fail it.
- **F7** (card 8), fixed as proposed.
  The board way forward now reads "run `lyx config <module> --layer default` from the hub's prime worktree", the form at `reviewbudget.go:97` and `configcli.go:533`.
- **F8** (card 10), fixed as proposed.
  - Added the Edit target `docs/overview.md` after `internal/configcli/doc.go`.
  - The Intent says the config bullet's `--layer default` sentence states three things:
    - it seeds absent board files sparse;
    - it prunes but never fills a present one;
    - it lists the one-time base strip as pending, which only a start verb's hub reconcile performs.
  - The sentence keeps the `--apply` commit clause.
- **F9** (card 9), fixed as proposed.
  - The Intent says `CloneAndWire`'s stamp-gated walk keeps `Ensure` and its failure, and gives the reason.
  - The ImpactSummary names the four call sites as the start verbs and the Preflight row, and says the clone's walk keeps `Ensure`.

## Self-check

- fix-affected-set, LOW: card 10 left the twin clause at `internal/configcli/doc.go:76` stale, "through the call the hub reconcile after a binary change makes". Fixed as proposed: card 10's doc.go paragraph now names that clause and replaces it.
- fix-proposal, LOW: F3's new exported `DropTemplateEqualNames` was missing from `internal/yamlengine/doc.go`'s function list and Per-name maps section, and had no test in its own package. Fixed as proposed:
  - card 6 adds the Edit target `internal/yamlengine/doc.go` with both doc changes;
  - card 6 adds the Create target `plan:internal/yamlengine#TestDropTemplateEqualNames`, a table-driven test in `layers_test.go`.
- fix-consistency: no findings.
- fix-claims: no findings.

## Disputed

None.

## Deferred

None.
