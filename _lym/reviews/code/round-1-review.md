---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/claudeengine/claudeengine.go:196
    summary: Prepare's wiring of the creel hook, hub board directory to flag directory and none when standalone, has no test.
    fix: Add a case to the existing Prepare tests in internal/shuttleengine/claudeengine that builds the engine once with a board directory and once without, and reads the settings file Prepare writes. Assert the hub case carries one PostToolUse entry with no matcher whose command names CreelFlagDir of that board directory, and the standalone case has no PostToolUse key.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/creelengine/server.go:81
    summary: The daemon-start ReconcileFlags call in Run is asserted by no test; deleting it breaks nothing, though an acceptance criterion names it.
    fix: Add a row to the Run or daemon-rig tests in internal/creelengine/server_test.go that leaves a stale flag file in Geometry.FlagDir and a missing flag for an unread strand before starting the daemon, then asserts both are put right once it serves.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/creelengine/deliver.go:296
    summary: The busy path of typeNotice (releaseNotice makes the notice claimable again), typeNotice typing nothing when a claim took the notice first, and the flag sync after a do whose fn journaled then failed have no test.
    fix: In internal/creelengine/deliver_test.go add rows beside NoticeBeingTypedIsNotClaimedAtABoundary - a busy Send, after which ClaimAtBoundary hands the notice out and the holder's wait is answered, and a notice claimed before the probe, for which Send is never called. In flags_test.go add a row where a do callback appends a record and then returns an error, and assert the flag still follows the pool.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/creelengine/flags_test.go:1
    summary: TestStore_MailFlags has no return row although the acceptance criterion names returning the last unread message as a flag-removal path.
    fix: Add a row to TestStore_MailFlags in internal/creelengine/flags_test.go that returns the strand's last unread message and asserts its flag file is gone.
  - id: F5
    severity: NIT
    class: design
    origin: lens:code-correctness
    location: internal/creelengine/verbs.go:188
    summary: The 8 KiB claim budget sums MessageLine lengths without the newline hookRead joins them with, so a claim can pass the budget by one byte per extra message, and no test row sits at the budget boundary.
    fix: In ClaimAtBoundary count one more byte per line after the first (size plus the joining newline) against boundaryClaimBudget, and extend StopsAtTheBudgetButHandsOutAtLeastOne in verbs_test.go with a row whose lines fit the budget only without the newlines.
---

### [LOW] Prepare's creel hook wiring is untested

**Location:** internal/shuttleengine/claudeengine/claudeengine.go:196
**Issue:** The criterion that a hub Claude strand's settings carry the hook and a standalone one does not is pinned only at `buildSettings`, which receives the flag directory as an argument. Nothing proves `Prepare` derives it from the board directory or converts it on Windows.

### [LOW] Daemon-start reconcile is untested

**Location:** internal/creelengine/server.go:81
**Issue:** `flags_test.go` calls `ReconcileFlags` directly, and no test through `Run` fails if the call is removed, so the acceptance criterion that a daemon start reconciles a stale or missing flag is unpinned at the daemon.

### [LOW] Pane reservation failure paths and sync after a failed fn are untested

**Location:** internal/creelengine/deliver.go:296
**Issue:** Only the happy reservation has a test. The release after a busy send, the yield to a prior claim and the sync after a journaled-then-failed `do` are behaviour the diff adds with no coverage.

### [LOW] No return row in the flag table

**Location:** internal/creelengine/flags_test.go:1
**Issue:** The post, read, forward, withdraw and program rows exist; the return path the acceptance criterion lists has none.

### [NIT] Budget ignores the joining newlines and is not pinned at its edge

**Location:** internal/creelengine/verbs.go:188
**Issue:** The decision record bounds lines to 8 KiB; the joined output is a few bytes longer than the counted sum. The focus asked to settle this: it is a literal-reading difference with a bounded error, not a defect that loses or duplicates mail.

## Rejected

- Code-correctness LOW, a failed `ReconcileFlags` at start leaves the hook without flags: the decision record makes flags derived state and the idle pane path still delivers, so the degradation is by design and logged.
- Code-correctness NIT, a flag file deleted outside the daemon is not recreated until restart: the record names the daemon the only writer of the flags, so outside edits are outside the contract.
- Double delivery or loss between `reserveNotice`, `releaseNotice`, `markNoticeRead`, `ClaimAtBoundary`, `Ack` and `Release`: traced by the handler and the correctness lens; `unreadLocked`, the pane probe and `Read` all treat `paneReserver` as reserved, so no defect.
- Empty claim printing an empty context: `Client.exchange` calls the handler only when `AckRequired` is set, so nothing is printed; no defect.
