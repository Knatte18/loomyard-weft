```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test row that pins it, or none.

**Per behaviour: pinning test**

1. `RunCapture` sets `WroteNoFile` on exit 0 with no file, with no skips.
   - Pinned by `TestRunCapture` row "exit 0 with no file fails" (`capture_test.go:109`, `wantWroteNoFile: true`, check at `capture_test.go:138`).
   - Other rows (pass, exit 3) assert `WroteNoFile` false through the default.

2. The reason under skips names the scope, the count and "may have been skipped" (plural).
   - Pinned by `TestRunCapture` row "exit 0 with no file under skipped tests…" (`capture_test.go:110-114`), which asserts "scope partial skipped 2 tests" and the "may have been skipped" text.

3. The reason has no skip text when nothing is skipped.
   - Pinned by the `capture_test.go:130` check, which applies to every row without `wantNamed`.

4. The singular wording for one skipped test ("skipped 1 test") in `noFileReason` (`capture.go:146-149`).
   - No test pins it.
   - `baseline_test.go:409` writes "skipped 1 test" into a hand-built `runRecord`, so the string never passes through `noFileReason`.
   - The only `RunCapture` skip row uses two tests.

5. Drift-Await replaces the plain capture way forward on a wrote-nothing record at the branch point.
   - Pinned by `TestDriftAwait` row "a record that wrote nothing under skipped tests is Stuck naming the scope changes…" (`await_test.go:84-92`). It asserts the rescope, scope-off and defer paths are present and that the plain capture path is absent.

6. Drift-Await, off the branch point, offers only scope off and defer.
   - Pinned by `TestDriftAwait` row "…with HEAD moved names scope off and defer only" (`await_test.go:93-103`).
   - It asserts `--scope full` and `lym drift capture` are absent.

7. A plain failed capture and a missing capture keep their old ways forward.
   - Pinned by the existing `TestDriftAwait` rows, which use `capturePath` (for example `await_test.go:81-82`).

8. The Drift-Baseline foreground-fallback Stuck names the rescope ways in place of a plain capture (`baseline.go:221-225`).
   - Pinned by `TestDriftBaseline` row "a foreground capture that wrote nothing under skipped tests…" (`baseline_test.go:405-418`), through `wantReason` and `wantNoReason`.

9. `emptyUnderSkips` returns false when `WroteNoFile` is true but `Skip` is empty (a failure with no skips keeps the plain capture way).
   - No Await or Baseline row covers this.
   - No row has `WroteNoFile` with an empty `Skip`.
   - `await_test.go` has no `WroteNoFile` record without skips.
   - The `baseline_test.go:409` fixture always carries `Skip`.
   - If the `len(Skip) > 0` condition were dropped, no test would fail.

10. `emptyUnderSkips` returns false when `Skip` is non-empty but `WroteNoFile` is false (for example "exit 3" under skips, or an old record read from disk).
    - No row covers this.
    - No Await or Baseline row has a failed capture with `Skip` set and `WroteNoFile` false.
    - If the `WroteNoFile` condition were dropped, no test would fail.
    - This is the old-record and other-failure case the focus asks about.

11. The `wrote_no_file` JSON key round-trips through `internal/state`, and an old record without the key reads as false.
    - No test.
    - The Await and Baseline tests build the record in memory. `writeCapture` in the Await fixture may go through `WriteCapture`, but no row reads back an old record without the key.
    - The risk is low, since it is a plain struct tag.

**Findings**

- **LOW, design (test gap)**
  - Location: `internal/driftengine/capture.go:146-149`, `capture_test.go:110`.
  - The singular-wording branch of `noFileReason` (one skipped test) is uncovered.
  - Fix: add a row with `skip: []string{"TestA"}` and `wantNamed: ["scope partial skipped 1 test,"]`, or similar.
  - Evidence: `capture_test.go:110-114` uses two skips only. `baseline_test.go:409` is a stub string.

- **LOW, design (test gap)**
  - Location: `internal/driftengine/capture.go:154-156`, `await_test.go:84-103`, `baseline_test.go:405-418`.
  - The two halves of `emptyUnderSkips` are never separated by a negative case:
    - `WroteNoFile` with empty `Skip`.
    - Skips with `WroteNoFile` false (another failure, such as exit 3, or a pre-change record).
  - Either missing condition would send a capture that failed for another reason to the rescope Stuck, with no test failing.
  - Fix: add one `TestDriftAwait` row, for example `Skip` set, `WroteNoFile` false and a failed record, that expects `capturePath` and no `--scope`. Optionally add a `WroteNoFile` row with no skips.
  - Evidence: all new Await and Baseline records set both fields.

I raised nothing on test economy:
- The new rows are distinct behaviours: on and off the branch point, foreground fallback, and the reason under skips.
- The `capture_test.go` assertions fold into the existing table.
- Nothing duplicates an existing test.

Tool calls: none; my context already held every file I needed.
