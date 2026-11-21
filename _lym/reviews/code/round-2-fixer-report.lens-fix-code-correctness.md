```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the F15–F18 change set through the correctness lens.

No BLOCKING or MEDIUM findings.

Walk-throughs:
- **`darnTable` closure (F15):** a gated writer loads the gate config and `MeasureCall` fills `measureCall`. An ungated writer skips both and passes `MeasureCall: ""`. The inner `err` shadow inside `if gateFixed` is harmless, since only `measureCall` is assigned to the outer scope.
- **`DarnTable` (F15):** `MeasureCall` is required only when `GateFixed` is true. The `makeChangeList` fill of an unused `measure_call` marker with `""` is safe, and the ungated spec row passes. The other caller, `shuttleparity/seats.go:212`, still sets a stand-in, so it holds.
- **`tiersRunBy` (F17):** for tier0 (`{targets} {args}`) and tier4 (no targets, `{args}`, covers `[0,2]`), a key listing `[0,4]` reduces to `[4]`. `MeasureCall` then renders `... -- <args>` with no target word.
  - It mirrors `dropCovered`'s descending walk, and `Gate()` has already checked the tiers are declared.
- **Tests fail with the defect in place:**
  - The `covered` row gets a target word without the reduction.
  - The `coverednoargs` row is refused without the reduction.
  - The `fixerOff` wiring row and the ungated `DarnTable` row fail without the conditional.
  - The gated no-`{args}` row's expected text matches the refusal message.
- **F18:** `trxLogFilePrefix` is `"gate"` (`trx.go:20`), so the literal `trx;LogFilePrefix=gate` matches. The tier1 spawn test was not in the gate run I saw, but the words are identical to `vstestResultsArgs`.
- **F16:** `resultsFileLines` and `removeResultsDir` are still used in production (`tierrun.go:160, 517, 570`), so nothing is left dead. The `io` import was removed and the other imports are still used.

Findings:
- **LOW**: `internal/loomcli/wiring_test.go`, row "no gate.yaml wires … and an ungated darn table still builds".
  - Summary: the row's old assertion, that an absent `gate.yaml` fails a gated darn table build with "gate config absent", was dropped. Nothing covers that `gateslot.LoadConfig` error path in `darnTable` now.
  - Evidence: the `wiring_darn_test.go` measuring-call rows cover `MeasureCall` refusals but always write a gate config. The `wantTableErr` field and its branch were removed outright.
  - A gated row with no `gate.yaml` would restore the coverage.

Calls made:
1. `grep` over `internal/gatecli` for `trxLogFilePrefix`, `removeResultsDir` and `resultsFileLines` outside `trx_test.go`. My context lacked `trx.go` and `tierrun.go`, which F16 and F18 depend on.
