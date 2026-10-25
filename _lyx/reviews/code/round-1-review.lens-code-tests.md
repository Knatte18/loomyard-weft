```yaml
lens: code-tests
outcome: reported
```

**Task:** code-tests lens. Per behaviour the diff adds or changes, the test that pins it, or none.

**Behaviour → pinning test**
- One suite start per call, after the resumes and the dobbies, however many entries were adopted: `TestAdopt/DobbiesThenOneSuiteStartAfterTheResumes` (`adopt_test.go:505`). It asserts the `w.order` sequence and `got.Suite == testSuite`. It replaces the old `DobbyStartRunsLastAfterTheResumes` row and keeps its resume-then-dobby order assertion, so nothing is lost.
- No owner recorded means no suite and no warning: `CallThatRecordsNoOwnerStartsNoSuite` (`:513`). It covers the stopped-slug case, and a refused adopt returns before the seam.
- A failed start sets the `suite_warning` text, leaves `Suite` nil and leaves the slug adopted and resumed: `FailedSuiteStartIsTheWarningNamingTheGate` (`:518`).
- The hub seam starts nothing under `go test`, and leaves no log directory: `TestStartSuiteStartsNothingUnderTest` (`deps_test.go:59`).
- The `StartSuite` branch condition `len(adopted) > 0 || result.PrimeRecorded` (`adopt.go:229`): only the `len(adopted) > 0` side is pinned, so the `PrimeRecorded` side has no test of its own (F1).
- The `suite` and `suite_warning` fields in the `cli.go` envelope, and the `pid` and `log` JSON tags on `suiteStart` (`adopt.go:64-67`): none (F2).
- The `startSuite` spawn itself (argument list, `Dir`, log truncation, the `logger` calls): none. It is unreachable under `testing.Testing()`, as for the other detached spawns, and needs no finding.
- `lyxdirs.AdoptDirName` and the log path built in `deps.go`: none. It is a constant and a join with no behaviour of its own.

**Test economy**
- No new test duplicates another test's coverage: the three new rows sit in the existing `TestAdopt` table, and the guard test is the only one of its kind.
- No per-helper test is redundant. `TestStartSuiteStartsNothingUnderTest` pins the acceptance criterion "starts nothing under `go test`", which `TestAdopt` cannot reach because it fakes the seam.
- No new assertion targets a stencil's static prose.

**Findings**
- **F1 — LOW, `scope`, `internal/adoptcli/adopt_test.go:505`.**
  - **Summary:** The `|| result.PrimeRecorded` branch of the suite-start condition (`adopt.go:229`) is not independently pinned.
  - **Evidence:** The new row passes `Prime: true` together with two slugs, so `len(adopted) > 0` is already true. A prime-only adopt (`request{Prime: true}`, no slug adopted) would still start the suite if that branch were deleted, and no test would fail. The decision record names "or of the prime" as an owner-recording case.
  - **Fix:** Add a prime-only assertion to the existing row or a sibling row, with `req: request{Prime: true}` and an empty `adopted`, checking `got.Suite != nil` and that `w.order` ends in `suite`. The `OwnerHaltedChildResumesWhenThePrimeIsAdopted` fixture (`:399`) already uses a prime-only request.
- **F2 — LOW, `scope`, `internal/adoptcli/cli.go` (new `suite` and `suite_warning` field blocks) and `internal/adoptcli/adopt.go:64-67`.**
  - **Summary:** No test pins the CLI envelope keys `suite` and `suite_warning`, or the `pid` and `log` JSON tags. Acceptance criterion 1 says the envelope carries the pid and log.
  - **Evidence:** `adopt_test.go` asserts the `outcome` struct only. I did not find a `cli_test.go` for the adoptcli package. I could not confirm whether the existing `dobby_warning` envelope field is tested anywhere, so this may be the same gap as for that field.
  - **Fix:** If an envelope-level test exists for `dobby_warning`, extend it with `suite` and `suite_warning`. Otherwise record this as a known gap, since the `adoptcli` package's convention is evidently to test the `adopt` function and not the cobra wiring.

**Tool calls**
1. `grep` of the row names in `adopt_test.go`, and of `dobby_warning|suite` in `cli_test.go` (the file does not exist). My context did not hold the full existing row list or any envelope-level test.
