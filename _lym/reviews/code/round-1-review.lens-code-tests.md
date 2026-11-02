```yaml
lens: code-tests
outcome: reported
```

**Behaviour → pinning test**

1. **`verify_interactive` loads from drift.yaml:** `config_test` row "verify_interactive loads" (`config_test.go:136`). Pinned.
2. **Key is classified local-open:** `configreg_test.go:80` key list. Pinned.
3. **`VerifyTable` sets `Table.Interactive` and swaps the mode and judging text:** `seats_test` "verify_interactive makes the verify lead the operator's session" (`seats_test.go:235`). Pinned: it checks `table.Interactive` and the prompt text in both modes.
4. **Parent directive drops its operator-ban line when interactive:** no assertion in the diff's tests.
   - The ban text is not checked absent from the rendered prompt in `seats_test.go:247-248`.
   - Only `table.Interactive` is asserted; the drop itself is `parentdirective.Directive`'s existing behaviour.
   - LOW; the seatengine and parentdirective tests may cover the rule, and I did not read them.
5. **Stencil markers `mode_rules` and `judging_rules`:** `markers_test.go:56`. Pinned.
6. **Autonomous text renders as before:** `seats_test.go:247`. Pinned, but only through two substrings. The unchanged stencil text is not compared with the old rendering.
7. **`ResetRuns` records the mode, and `OperatorSessionOpen` reads it:** `records_test` `TestTakeRun` (`records_test.go:64-99`). Pinned for both `false` and `true`.
8. **`CloseAgentSession` makes `OperatorSessionOpen` false:** `records_test.go:110-114`. Pinned. The `Operator` field stays true after close, so only the `&& SessionOpen` conjunct is exercised.
9. **`runAgent`'s `prepare` passes `cfg.VerifyInteractive` to `ResetRuns`:** `verify_test` "an operator session's acceptances are ruled…" (`verify_test.go:772`). Pinned.
   - If `prepare` passed `false`, `sessionAccept` would record the acceptance as agent and unruled.
   - The test would then fail: the row would end Stuck instead of Done, and `By` and `Ruling` would not match.
10. **`Accept` records `by: operator`, ruled under scope full, under partial scope:** `verbs_test` rows at `verbs_test.go:138-139`. Pinned.
11. **Operator session may re-accept a withdrawn test:** `verbs_test.go:140`. Pinned, including the withdrawn list shrinking.
12. **An autonomous agent is still refused a withdrawn test:** `verbs_test.go:145` and the Withdraw subtest at line 198. Pinned.
13. **Operator mark with the session closed is the row's own work:** `verbs_test.go:141`. Pinned.
14. **Done-time unruled check and the decision-halt path still treat agent acceptances as before:** existing agent-under-full rows (`verbs_test.go:137`, existing `verify_test` halt tests). Pinned. The diff leaves those paths unchanged.
15. **Operator-ruled acceptance does not raise a second halt:** `verify_test.go:772`. Pinned.
16. **Resumed live agent keeps the mode it was spawned with:** **none**.
    - The summary and the doc claim it.
    - It holds only because `prepare` is skipped on resume.
    - The resume test "a live chair is resumed before any check" (`verify_test.go:628`) seeds no operator `ResetRuns` and asserts no `Operator` or `SessionOpen` value afterwards.
    - No row checks that a resume neither rewrites `runs.json` nor changes the mode when `verify_interactive` is flipped between spawn and resume.
17. **`driftcli` `verbStatus` reads `OperatorSessionOpen` into `VerbStatus.OperatorSession`:** **none**.
    - My grep over `internal/driftcli/*_test.go` for `verbStatus`, `OperatorSession` and `verify_interactive` found no match.
    - Only `capture_integration_test.go:221` changed, for the new `ResetRuns` signature.
    - If the wiring dropped the field, every `driftengine` test would still pass, and operator acceptances would silently record as agent and unruled.
18. **Help text of `accept` and `withdraw`:** static prose, exempt.
19. **Interrupted entry that finds no live chair closes the session:** the existing test "no live chair closes the mark and goes on to the compare" (`verify_test.go:648`) covers the close. No row for the mark after an interrupted operator session.
    - It is covered at the `records_test` level only, so this is an uncovered combination, not a missing behaviour.

**Findings**

- **MEDIUM, class `design`:** the wiring that carries the operator mark to `lym drift accept` has no test.
  - Location: `internal/driftcli/wiring.go:69-77`, the `OperatorSessionOpen` read in `verbStatus`.
  - Evidence: no `driftcli` test references `verbStatus` or `OperatorSession`. The only `driftcli` test change is the signature update at `internal/driftcli/capture_integration_test.go:221`.
  - The `verify_test` `sessionAccept` helper (`verify_test.go:205-222`) re-implements the wiring instead of calling it.
- **LOW, class `design`:** the "resumed live agent keeps its spawn mode" claim in `_lym/landing/summary.md` and `doc.go` has no test.
  - Location: `verify.go:123-130` (the resume path skipping `prepare`) and the existing test at `verify_test.go:628`.
  - A resume test with `ResetRuns(…, true)` seeded and `VerifyInteractive` flipped to false would pin it.
- **LOW, class `design`:** nothing asserts the parent directive's operator-ban line is absent from the interactive verify prompt.
  - Location: `seats_test.go:247-248`. The `absent` list holds only `autonomousVerifyMode` and the "Present the drift to the operator" check.

**Tool calls:** 1 (`grep` over `internal/driftcli/*_test.go` and `verify_test.go`), because the manifest did not show the `driftcli` tests or the resume-path rows.
