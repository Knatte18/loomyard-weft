---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: tools/tokencount/count.go:1023
    summary: The All runs section always prints that every time figure leaves out machine sleep, even under -no-sleep or with no journal, where the header says sleep counts as time.
    fix: In WriteMarkdown, print the "Every time figure leaves out machine sleep" line only when slept is greater than zero, as the per-run line already does, or word it so it does not claim a subtraction that did not happen. Extend the existing sleep assertions in TestShedRecordSpanAndProducers with a case where no sleep was subtracted and the line is absent.
  - id: F2
    severity: LOW
    class: consistency
    location: tools/tokencount/sleep.go:25
    summary: The degraded journal paths (journalctl absent, failing, parse error, no sleep recorded) and their header lines have no test, though the decision record's last acceptance criterion is that the header names why no sleep was subtracted.
    fix: Split the outcome mapping out of journalSleeps into a pure function taking the journalctl output and its error, for example sleepsFromJournal(out []byte, err error) ([]Interval, string). Add rows to TestParseSleepJournal, or a table in the same file, asserting the header line for a failing command, an empty journal and a parse error. journalSleeps keeps only the LookPath and exec calls.
---

### [LOW] All runs claims a sleep subtraction that did not happen

**Location:** tools/tokencount/count.go:1023
**Issue:** The line `Every time figure leaves out machine sleep, 0s over these runs.` is written for every report.
With `-no-sleep`, on Windows or with an empty journal, the header says sleep counts as time while this line says it is left out, so the two disagree.
The per-run line is guarded by `slept > 0`, so the two sections also disagree with each other.

### [LOW] Journal degradation path is untested

**Location:** tools/tokencount/sleep.go:25
**Issue:** The board entry's design and the decision record require a missing or failing journal to degrade to a header line, never an error.
The code does this correctly on reading: every failure returns `nil` and a note.
Only the `-no-sleep` line is pinned, in `TestRunReadsArchivedShedStatus`; the notes for `journalctl` absent, a failing command, a parse error and a journal with no sleep have no test.
Because `journalSleeps` runs the real command, a test cannot reach those branches without mutating `PATH`, which the parallel tests forbid.

## Checked and found sound

- `subtractSleeps` and `withoutIntervals`: a working interval loses exactly the sleeps' union.
  Working excludes sleeps before `Idle()`, `Producers()` and `RoleTime()` read it, so idle is never negative and nothing is subtracted twice.
  The `Roles` map holds pointers, so the per-role subtraction persists.
- The journal parser pairs by operation and boot, and a return with no start, a start with no return and a pair split across boots yield nothing.
  The real journal on this host emits the fields it reads.
- Producers share is active over summed active, ordered by active time then name, as the entry asks; the row expectations and the share figures in the tests match.
- `roleOf` drops `-chair` after the `-N` suffix only, so advisor roles are not folded; the one-line comment and the `doc` text match the code.
- `-no-sleep` is licensed by the decision record as a hermetic-test switch, and the header lines follow from the board entry's "subtract hibernation" and the decision record's missing-journal rule.
- The test-budget bump of one matches the one new top-level test, `TestParseSleepJournal`; the other new coverage extends existing tests.
