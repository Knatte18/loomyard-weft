```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I name the test that pins it, or none.

**Per behaviour, the pinning test**
- **`depends-note` check fires for a part depending on a board note.** `TestCheck/DependsOnNote` in `internal/raddleparser/check_test.go:77`. It would fail if the check were removed. It would also fail if the note fell through to `depends-unknown`, since the row asserts exactly `[CheckDependsNote]`.
- **`depends-note` fires while filing, even with the note in `board_checked`.** `TestCheck/DependsOnNoteWhileFiling` at `check_test.go:81`.
- **A `depends_on` naming both a part and a note counts as the part (summary design decision).** None. The `isPart` case in `check.go:376` has no row.
- **The `depends-note` Detail text, with the way forward and the full `lym board promote '{"slug":"…"}'` command.** None. `TestCheck` compares only check names. The text is the refusal the operator and agents read, and the diff's own purpose is to complete it.
- **The board refusal names both commands in full, with the remaining dependencies.** `TestKindDependencyRules/depending on an open note refused` in `internal/boardengine/store_test.go:137`, an exact-string match. It passes `dep` and `idea`, so it pins removal of the noted dependency. The sole-dependency case, which renders `[]`, has no row.
- **The child's stdout is written to the loop log with a `lym shed: step <trace id> output: ` prefix per line.** `TestLoop_ChildStdoutIsMarkedInTheLoopLog` in `internal/shedverbs/loop_test.go:1462`. It covers the prefix and the newline guard for an unterminated prior log line. It also covers the last line having no trailing newline.
- **Empty child stdout appends nothing (`appendLoopChildOutput`, `loop.go:758`).** None. The test always writes non-empty output.
- **Child stdout that ends in a newline keeps no extra empty prefixed line (the `TrimSuffix` in `loop.go`).** None. The test input is `"first\nsecond"`.
- **The dead-loop excerpt leaves child output out and keeps the loop's own lines.** `TestUntilStop_DeadLoopStopsLoopExited/ChildOutputLeftOut_TheLoopsOwnLinesExcerpted` at `loopwait_test.go:599`.
- **An unrecorded exit says there is no start mark.** The rows `NoTraceID_LoopLog` and `NoStartingMark_OnlyTheLoopExitClause`, plus the updated `AfterInterrupt` assertion at `loopwait_test.go:947`.
- **An unrecorded exit with a start mark says no process marked the exit.** `StartingMarkWithNoExitMark_SaysWhyNotRecorded` at `loopwait_test.go:615`.
- **`loopLogLinesOf` returns `started`, and `loopProcessExitRecorded` is adapted.** Both are covered through the dead-loop stop test rows. This is an internal helper and needs no test of its own.
- **Fresh-start behaviour: with no loop output of its own, no `loop log:` clause appears.** This is unchanged. The rows I saw carry no `loop log:` clause where the loop has no output.
- **Doc text in `doc.go` and `raddle-proposal-spec.md`.** Static prose, exempt.

**Findings**

1. **LOW, class `scope`.** The `depends-note` Detail (the way forward and the promote command) is unasserted.
   - Location: `internal/raddleparser/check.go:378`; test `internal/raddleparser/check_test.go:77`.
   - Fix: add a Detail assertion for the `DependsOnNote` row, or a small test on `Check` output. It should assert the note slug, the drop-the-dependency way forward and the full promote command.
   - Grounding: `PATTERN-refusal-way-forward` says a new refusal lands with a test asserting its message.
   - Caveat: `raddleparser` is outside the packages that entry binds (webster, shed, loom). The check is surfaced through the loom raddle verbs, so I rate it LOW.

2. **LOW, class `scope`.** The part-versus-note precedence (`isPart` wins over a note) has no row.
   - Location: `internal/raddleparser/check.go:376`.
   - Fix: add a `TestCheck` row where a part slug also appears as a `KindNote` board entry in `b.Entries`, expecting no `depends-note` finding.

3. **LOW, class `scope`.** `appendLoopChildOutput`'s empty-stdout no-op and its trailing-newline trimming are unpinned.
   - Location: `internal/shedverbs/loop.go:758`; test `loop_test.go:1462`.
   - Fix: make `TestLoop_ChildStdoutIsMarkedInTheLoopLog` table-driven. Add rows for empty stdout (the log stays unchanged) and stdout ending in `\n` (no empty prefixed line). The existing case becomes a row too.

No test-economy findings. The new rows extend existing tables without duplicating a row. The one new test pins a behaviour no other test covers. The four test-economy candidates in the redundancy report were all listed as "not judged".

**Tool calls:** none. My context already held the diff and the test files.
