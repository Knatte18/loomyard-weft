```yaml
lens: code-affected-set
outcome: reported
```

Task: for each signature, output, key or message the diff changes, find who reads or asserts it, and report any member the diff leaves wrong.

**Affected set per change**
- `gateslot.FailedTier` now returns `(tier, build, ok)`. Its only production caller is `internal/landingshed/verifygate.go:84`, updated. The test at `internal/gateslot/runlog_test.go:185` is updated. Covered.
- `tierFingerprint(tier, build, inert)` and `openGreen(..., cfg)`. Callers are `green.go:246`, `green.go:267`, `run.go:70` and `green_test.go`, all updated. Covered.
- `renderTierCommand(number, tier, targets, terms, args)`. Callers are `tierrun.go:138` and `tiers_test.go:575`, updated. Covered.
- `impactset.resolveAncestorBase` is now `AncestorBase`, and `changedPathsInTree` is now `ChangedPaths`. A grep finds no stale reference. Covered.
- The `scoper` value set gains `dotnet`. The `config.go` refusal text, the `template.yaml` comment, `gateslot/doc.go:72` and the `tiers.go` `--guards` refusal are all updated. Covered, except the doc line in F1.
- The build timeout line and `FindTierTimeout`. `verifytree/verify.go:481` builds the Detail `gate build outlived…`, which matches the `gate build ` prefix that `landingshed/verifygate.go` tests in `verdict`. Covered.
- `LandingFailure.Build` (new `build: true` key). The producer is `verifygate.go`. `contracts/stencils/landing/landing-template-fixer.md` gained a paragraph that reads the key. I did not search for other stencils that render the failure record. Covered for the stencil I found.
- Run-record file name `<stamp>-build-<pid>`, the new `Build` and `BlockedBy` fields, and the `build_runs` and `build_seconds` report columns. `report.go` and its docs are updated. I found no other reader of `RunRecord` in production code. Covered.
- Docs: `docs/dotnet-hub.md`, `docs/overview.md`, the `doc.go` files and `PATTERN-gate-slot.md` are in the diff. Covered.

**Findings**

- **F1, LOW, consistency.** `internal/gatecli/doc.go:181` says "under the `go` scoper a package entry covers its packages, so a remainder runs as explicit packages; under the `none` scoper only a whole entry carries". The `dotnet` scoper now also carries only whole entries. `recordPass` skips `inv.partial` invocations, and the decision record's Constraints require this. The sentence names only `none` and leaves `dotnet` unmentioned. Fix: name `dotnet` beside `none` there.

I found no other stale member of the affected set.

**Tool calls**
1. A grep of Go files for the changed symbols' callers, plus `git diff --stat` and `ls docs`. My context lacked the caller lists and the file list.
2. A grep for `scoper` value mentions and `RunRecord` readers, and a diff of the fixer stencil. My context lacked the stencil diff and the other scoper mentions.
3. A grep of `FindTierTimeout` and its `verify.go` consumer. My context lacked `verifytree`'s handling of the build timeout line.
