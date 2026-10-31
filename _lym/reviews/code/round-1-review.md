---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/adoptcli/deps.go:238
    summary: adoptGateTiers and the StartSuite wiring that carries Tiers on suiteStart have no test, so D1's tiers-left-out acceptance criterion is unpinned
    fix: Add a tier0 test in internal/adoptcli/deps_test.go of adoptGateTiers over a prime fixture with three cases - a gate config holding an adopt key (returns its tiers), a gate config without the adopt key (returns nil), and a gate config that does not read (returns nil) - so removing the Tiers assignment in hubDepsAs's StartSuite, or a panic on the missing key, fails a row.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/adoptcli/check_test.go:47
    summary: The fixture clock is constant, so a mark timing the tier in flight from the check's start instead of the tier's own start would still pass TestRunCheckMarks
    fix: In newCheckFixture make Now advance on each call (for example by one second), and in TestRunCheckMarks assert that the Start of each tier-in-flight mark differs from the starting mark's Start and equals the time the tier started.
  - id: F3
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/adoptcli/cli.go:105
    summary: The envelope emitting message as a JSON field is asserted nowhere, since TestAdopt checks only the outcome struct
    fix: Add a case to a test that runs the adopt command and decodes its one-line envelope, asserting message is present after a started check and absent when suite_warning is set; if the package has no command-level test, add a small tier0 one that runs the command over the fake deps.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/reedengine/bar_integration_test.go:317
    summary: The new bar test pins the mark only as after the follow button, so a mark rendered after the clock would still pass
    fix: In TestBar_LineTwoShowsTheCheckMarkBeforeTheClock assert that the index of the mark text is less than the index of the clock text in the expanded line, beside the existing follow-button comparison.
---

### [MEDIUM] adoptGateTiers and the tiers wiring are untested

**Location:** internal/adoptcli/deps.go:238
**Issue:** The diff adds `adoptGateTiers` and the `StartSuite` line that sets `Tiers` from it, which carry D1's rule that a gate config that does not read, or holds no `adopt` key, leaves the tiers out. `TestAdopt` feeds `Tiers` through its fake `StartSuite`, and `TestEnsureCheck` never sets them, so no test fails if the tiers are never carried or the missing-key path breaks.

### [LOW] The fixture clock cannot tell the tier's start from the check's start

**Location:** internal/adoptcli/check_test.go:47
**Issue:** `checkFixtureNow` is returned by every `Now` call, so `record.Start` and `tierStart` are equal in `TestRunCheckMarks`. Timing the tier in flight from the wrong start passes.

### [LOW] The envelope's message field is not asserted

**Location:** internal/adoptcli/cli.go:105
**Issue:** Dropping the `fields["message"]` assignment fails no test: `TestAdopt` asserts `outcome.SuiteMessage`, and nothing decodes the printed envelope.

### [LOW] The mark's place before the clock is not pinned

**Location:** internal/reedengine/bar_integration_test.go:317
**Issue:** D3 and its acceptance criterion put the mark before the clock. The test compares the mark's index only against `nolist align=right`.

## Rejected

- Lens code-tests F2 (a notice and a mark failing together): the row fails both and asserts the record is unchanged, which is the criterion; a split adds no behavior.
- Lens code-tests F5 (no row for a missing session in `SetCheckMark`): the session check is the first statement of the method and the same helper guards `SetWaitMark`; a row would test the helper again.
- Lens code-tests F7 (`hubCheckDeps` Mark and `primeReed` untested): the failure path is covered at `runCheck` by the failing-mark row, and the wrapper only forwards to `SetCheckMark`, which has its own tests.
- Lens code-correctness and code-affected-set: no findings; the handler's own reading of the diff against D1 to D3 (texts, order, envelope shape, display-only options, no new loop or process, docs landing) found no defect.
