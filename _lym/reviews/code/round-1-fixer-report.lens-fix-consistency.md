```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix change set for stale siblings or twins.

**Findings**

1. **LOW**, `_lym/landing/summary.md:67` (the design decision on unreadable or unparsed test files). It says a test file that cannot be read or parsed runs the package whole "with a `Go file does not parse` miss".
   - The F8 change in `internal/gatecli/coveragescope.go` (`importerPackage`) now labels a `GuardScanError` from `FindGuardTests` as `miss: impactset: <file>:<line>: …`, not the parse miss.
   - The label list in the same file (the Stderr bullet, `miss: <reason>: <detail>`) also has no form for that label.
   - Evidence: the `IsGuardScanError` case in `importerPackage` and the `labelPrefix` row "misplaced guard marker" in `coveragescope_test.go`.
   - The summary should name this second label.

2. **LOW**, `internal/gatecli/coveragescope.go`, doc comment of `narrowCoverage`. It reads "A selection miss, or a test file that cannot be read, runs the package whole." It does not mention a guard scan failure, which now carries its own label. The behaviour is the same (whole run), so only the wording is incomplete.

**Checked, no discrepancy**
- The three scope edits in `internal/gateslot/doc.go` (the Tier plan table) now agree with the doc's own line 103: "template gives it to `webster-fix`, `darn-write` and `code-burler-fix`".
- The `internal/gatecli/doc.go` sample block now shows `coverage` for those three keys. The other hits are tests with hand-written hub files, so `diff` is correct there:
  - `internal/gateslot/config_test.go`
  - `internal/websterengine/verifygate_integration_test.go`
  - `internal/loomshed/verifygate_integration_test.go`
  - `internal/configcli/configcli_integration_test.go:661`
- The `gateslot/doc.go` migration lines at 204–205 describe what migration emits (scope `diff`). They are deliberately unchanged.

Tool calls: 1 (one `grep` over `internal/gatecli`, `_lym/landing/summary.md` and the repo for stale `diff` scopes and "does not parse"), because my context lacked the twin sites outside the edited files.
