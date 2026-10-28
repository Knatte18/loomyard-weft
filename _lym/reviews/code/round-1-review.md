---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/reedengine/doc.go:485
    summary: doc comment names smoke_lifecycle_test.go as the home of TestSmokeRemoveLastStrandThenAddRunsTheNewCommand, which the diff moved to smoke_lifecycle_windows_test.go
    fix: In internal/reedengine/doc.go lines 485-486 replace the path internal/reedcli/smoke_lifecycle_test.go with internal/reedcli/smoke_lifecycle_windows_test.go.
    origin: lens:code-affected-set
  - id: F2
    severity: LOW
    class: consistency
    location: PATTERN.md:60
    summary: the new no-skip rule is stated only in pattern/PATTERN-test-speed.md; the PATTERN-test-speed entry line in PATTERN.md does not carry it
    fix: In PATTERN.md line 60 add a clause to the PATTERN-test-speed rule, such as "a skip names a missing prerequisite, never the OS", keeping the entry's single-line format and background link.
    origin: lens:code-affected-set
---

### [LOW] Doc comment names the moved test's old file

**Location:** internal/reedengine/doc.go:485
**Issue:** The reedengine doc cites `internal/reedcli/smoke_lifecycle_test.go` as where `TestSmokeRemoveLastStrandThenAddRunsTheNewCommand` verifies the psmux behavior. The diff moved that test into `smoke_lifecycle_windows_test.go`, so the cited path no longer holds it.

### [LOW] PATTERN.md entry omits the new no-skip rule

**Location:** PATTERN.md:60
**Issue:** CLAUDE.md puts a new cross-cutting invariant into PATTERN in the same commit. The diff adds the rule as a section of `pattern/PATTERN-test-speed.md` but leaves the `PATTERN-test-speed` rule line unchanged. The directive declares no PATTERN change, so this is only a consistency gap.

## Rejected

- Remaining `crosscompile_test.go` host check (`GOOS`/`GOARCH` with `CC` unset): a combined check whose operative half is a missing cross C toolchain, so a missing prerequisite, not a skip on the OS alone.
- Windows `go vet` error `undefined: mustTmuxOutput` in `internal/reedengine/contract_integration_test.go`: in files the diff does not touch, present at the base.
- Board note `verifygate-misplaced-guard-test` not visible in the diff: board writes go through `lym board` into the records repo, so the diff cannot show it; the change description claims it.
- `docs/benchmarks/test-redundancy.md` still names the deleted and removed tests: a measurement report of one run, not a claim to keep current.
- Inherited child variable in an unrelated nested test process, and two child variables set at once (D5 focus): only a hand-exported variable triggers the first, and the parent sets one variable per spawn; verdict on both is not a defect.
- Child spawned with `-test.run=^$` reaching TestMain's dispatch before `m.Run` (D5 focus): it does, and the exit code and stdout reach the parent as the cross-process and crashed-holder steps expect.
- Omitted-cell tally (D4 focus): still fails when ran or omitted disagrees with the tables, and no comment or doc.go text describes omitted cells as skipped.
- D2 row filters, D1 moves, D3 and D6 deletions: each drops the rows or tests the old skip skipped on the other OS and leaves no orphaned helper or import; every added file carries one tier tag plus a platform term, and Linux `go vet` over all touched packages is clean under tier0 to tier2.
