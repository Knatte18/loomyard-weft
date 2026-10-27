```yaml
lens: fix-consistency
outcome: reported
```

No findings.

Both changes agree with their surroundings:
- `internal/reportengine/doc.go:131` now lists Drift-Await in darn's setup. That matches `timingspans.go:106`, where `loomshed.NameDriftAwait` maps to `PhaseSetup`.
- The loom sentence at `doc.go:129` already names both Drift rows, so the two sentences are consistent.
- The `darn-recipe.yaml` header names `lym drift waive`, which exists in `internal/driftcli/cli.go:496`. `internal/driftengine/doc.go:94` says a missing counted capture is a decision halt that names `lym drift waive`, and that matches the header's wording.
- `docs/overview.md:498` and `:564` already describe Drift-Await in darn, so no twin was left stale.

Tool calls made (2 of 6):
1. `grep` of `timingspans.go` for Drift-Await and `lym drift waive`, and of `docs/overview.md` for Drift-Baseline. My context lacked the `timingspans.go` mapping and any check that the waive command exists.
2. `grep -i await` on `timingspans.go`, and Drift lines in `doc.go`. The first grep missed the symbol because the code uses the `NameDriftAwait` constant instead of the literal string.
