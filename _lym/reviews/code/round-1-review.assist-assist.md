---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/fabricengine/pushanchored.go:230
    summary: Transient post-push failures bypass owed-push classification and still halt Seed-IDE.
    fix: In pushanchored.go, preserve the bounded retry but record exhausted still-unpushed failures as deferred owed pushes and post-push transport failures as transport owed pushes through the existing failed-push writer and notifier. Preserve the underlying error and keep permanent errors and ended entries distinct, so IsPushOwed lets statuscommit retry transient failures without advancing the run.
    origin: lens:code-correctness
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/fabricengine/pushanchored_integration_test.go:523
    summary: The new loop has no regression rows for exhaustion or a failing post-push origin lookup.
    fix: Extend the existing anchored-push behavior test with rows that keep records unpushed through the backoff and that fail the origin lookup only after a successful push leaves commits unpushed. Inject the backoff sleep, assert the bounded attempt count and underlying error, and verify records failure classification, notification, IsPushOwed, and withheld code. Match transient failure expectations to F1 rather than pinning the current unclassified return.
    origin: lens:code-tests
---

### [BLOCKING] Transient failures still halt the transition

**Location:** internal/fabricengine/pushanchored.go:230

**Issue:** A successful push can leave another writer's commit unpushed on each attempt.
The exhausted loop returns its sentinel without a failed-push record.
A transport failure in the added origin lookup likewise returns an ordinary wrapped error at line 334.
Neither error carries the classification `IsPushOwed` requires.
Batten supplies that predicate at `internal/battencli/commitstatus.go:174`;
the transition seam returns a non-owed error at `internal/statuscommit/statuscommit.go:124`.
These transient states therefore still halt Seed-IDE, contrary to the board entry's Expected behavior.
They also bypass the failure record and notice required by `PATTERN-pair-remote`.
The affected-set lens independently found this same defect and is consolidated here.

### [MEDIUM] Failure outcomes of the added retry lack coverage

**Location:** internal/fabricengine/pushanchored_integration_test.go:523

**Issue:** The added hook creates one commit and the next push succeeds.
It never reaches the added loop's exhaustion branch or a failure of its post-push lookup.
Existing rejection-exhaustion and unreachable-origin rows exercise earlier classified-push paths.
They cannot detect a broken attempt bound, lost error classification, or code sent after these records failures.
The tests lens's two coverage findings are consolidated here.

## Focus assessment

The retry indexes delays only after checking exhaustion, including an empty or nil slice.
The nil sleep defaults to `time.Sleep`, and each attempt creates a fresh classified push job under the existing lock.
Fresh deferrals take precedence over origin lookup.
The lookup distinguishes an error from absence and requests the current branch's exact ref on origin.
One transient tracking-ref write failure can recover because the next push attempts that write again.
The synchronous post-receive hook exercises the concurrent-commit path without a timing wait.
It advances local HEAD before the first push returns, and the assertions require that advance to reach origin.
The old implementation would return its ended error on this path.
No duplicate test was identified; the redundancy report labels the test not judged.
The package documentation describes the new behavior, and the test header replaces its duplicated line.
All changes trace to the goal or its necessary tests and documentation.

## Rejected

- The tests lens's proposed assertion that a lookup failure permits no further push is too broad.
  Transient transport failures must remain owed under the board requirement; permanent lookup errors stay distinct.

## Validation

The focused tier-1 gate was attempted with the anchored-push behavior test selected.
It could not acquire a slot because the sandbox refused the holder-record write under `_bolt/.lym/gate`.
No tests ran, and no target files were changed.
