---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: internal/websterengine/recoverbatch_test.go:1570
    summary: TestRecoveryRetry has no row pinning the zero-count early return behind the kind and terminal checks
    fix: In the TestRecoveryRetry table add two rows built on deadRecovery(0, "") whose setup mutates the returned record, one with Kind "fork" and one with Terminal false, both wanting retry false and recoveries 0.
    origin: lens:code-tests
---

### [LOW] TestRecoveryRetry does not pin the zero-count branch's position after the kind and terminal checks

**Location:** `internal/websterengine/recoverbatch_test.go:1570`
**Issue:** The new `recoveries == 0` early return in `RecoveryRetry` is safe only because it runs after the kind, terminal, dead and cap checks.
No row has a zero-count record that is a fork or non-terminal, so moving the branch above those checks passes every current row.
The cap order and the dead status are pinned by existing rows.

## Rejected

- Stencil edit drops "that second recovery is the last" (pattern-fit note to the handler): the clause is false once a reset batch earns a fresh allowance, and the cap of two still bounds retries in code, so the removal follows from the amended rule and weakens nothing.
- `status` row duplicates the engine row (code-tests economy remark): it pins the `recoveries` and `recovery_retry` JSON fields and the no-git path, which the engine row does not.
- No `recover-batch` terminal-envelope row at zero recoveries: the envelope reads the same `RecoveryRetry` as `status`, so a row would restate covered coverage.
