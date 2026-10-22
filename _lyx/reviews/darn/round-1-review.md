---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    origin: handler
    location: internal/fabricengine/archive.go:21
    summary: The archiveTagPushDelays comment and the summary say the teardown holds no lock other writers wait on, but Remove runs the archive push under the pair's exclusive teardown lock
    fix: In internal/fabricengine/archive.go reword the second comment line of archiveTagPushDelays so it no longer says the teardown holds no lock other writers wait on; say instead that a teardown archives once and holds only the pair's own teardown lock, which only that pair's detached pushes wait on. Correct the matching sentence under "Design decisions" in _lyx/landing/summary.md the same way.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/archive.go:112
    summary: The archive push's own schedule (2 s doubling to 32 s, six attempts) and its use at the PushWithRetry call are pinned by no test, so reverting the call to PushBackoffDelays() or changing the delays breaks nothing
    fix: Add a row to an existing fabricengine unit test file that asserts archiveTagPushDelays() equals the sequence 2s, 4s, 8s, 16s, 32s, reaching it through export_test.go if the test package is external; the schedule is the operator-requested longer backoff and the only behavior of this change with no coverage.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gitrepo/pushretry_test.go:76
    summary: No PushWithRetry row covers an empty delays slice, the edge of the one-more-attempt-than-delays arithmetic
    fix: Add a row to the table in TestPushWithRetry_RetriesOnlyRetryableRefusalsUpToTheBound with delays nil, results of two retryable refusals, wantErr failed, wantExhausted true, wantPushes 1 and wantSleeps nil.
---

### [MEDIUM] The archive push's "no lock" rationale is false for Remove

**Location:** internal/fabricengine/archive.go:21
**Issue:** `Remove` takes the pair's teardown lock exclusive at `internal/fabricengine/remove.go:207` and calls `archiveWeftTip` at `remove.go:266`, so the push, with up to about a minute of backoff, runs under that lock for the whole retry.
The pair's detached pushes take that lock shared and wait on it or exit.
The code behaves acceptably, since the stall is bounded to one pair and a failed `Remove` respawns the push, but the stated reason for the longer schedule is wrong, and the code comment repeats it.
`Cleanup` also calls `archiveWeftTip` (`cleanup.go:247`) and its locking was not checked.

### [MEDIUM] The archive schedule has no test

**Location:** internal/fabricengine/archive.go:112
**Issue:** A search of the `fabricengine` tests for `archiveTagPushDelays` and `ErrPushRetriesExhausted` finds nothing, and the `gitrepo` tests pin only `PushWithRetry` and the plain primitives.
The longer backoff is one of the two changes the board entry asks for, yet a test passes with the old `PushBackoffDelays()` at the call.

### [LOW] No empty-delays row for PushWithRetry

**Location:** internal/gitrepo/pushretry_test.go:76
**Issue:** With empty delays the code makes one attempt and returns `ErrPushRetriesExhausted`, which is correct, but no row pins it.
The single-delay edge is held by the new "persistent refusal sleeps the caller's delays" row.

## Rejected

- Short-form line `v1` matching full ref `refs/tags/v1` cannot be told from a branch `v1`: inherent to git's printed output, and a line only matches the exact ref the caller passed.
- `namesRef` with a degenerate ref `refs/heads/` matching an empty name: no caller passes it.
- A negative delay would panic the default sleep's `rand.N`: callers control the delays and none passes a negative one.
- The `refs/tags/archive/...` integration test duplicating the unit rows: it pins that real git prints the short form, which the hand-written lines cannot.
- `docs/research/logging-inventory.md:80` citing a moved line: a dated report, not kept current.
- Focus questions on exact ref boundaries, the plain primitives' schedule and attempt count, the one-more-attempt arithmetic and the doc statements all held; the only departures found are F1 to F3.
