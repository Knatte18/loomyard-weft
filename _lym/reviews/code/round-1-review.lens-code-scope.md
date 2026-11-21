```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens. It traces each change in the diff to a goal point of the decision record, and each goal point to the changes that build it.

**Per change: the decision it traces to**

1. **TRX results arguments and parse.** `internal/gatecli/trx.go`; `runInvocations` in `tierrun.go`; `vstestCounter.startInvocation`, `replaceInvocation` and `RunMessages` in `counters.go`; the gatecli `doc.go` TRX paragraph. Traces to Goal #163, first fix: parse the result from the TRX file so the call's own loggers cannot change the verdict.
2. **Run-level messages (crash and the test running at the time).** Printed, logged and added to the causes in `runTierCommands`; `resultsFileLines`; `--blame` in `vstestResultsArgs`. Traces to Goal #163, second fix: capture the crash cause in the gate log.
3. **Gatecli `doc.go:109` rewording ("the counter reads" → "the line reads").** Forced by change 1: the counter now takes totals from the TRX file, so the old wording went stale.
4. **`gateslot.Config.MeasureCall` and its doc.** Covers falling back to `--tier 0` when `Gate` refuses the key, and adding the target word only when a tier takes targets. Traces to Goal #156 and #162: the prompt names only a form the config can run, with a fallback to `--tier`.
5. **Stencil `{{.measure_call}}` marker and the "every tier the call runs" wording.** Also `DarnInputs.MeasureCall`, the `makeChangeList` fill, the darnengine doc, the stencils.go doc, the marker set in `markers_test.go`, the darnengine `spec_test` rows, the `wireDarn` gate-config load with its `ErrConfigAbsent` fallback (commit `2b395b65`), and the shuttleparity stand-in. Traces to #156 and #162; these are the wiring, callers and tests that change 4 forces.
6. **`Config.Gate`'s unknown-key refusal lists the declared keys,** with its doc and `config_test` rows. Traces to #162: "have `lym gate test` refuse an unknown `--gate` with a list of declared keys".
7. **`envOverrideLine`, `testRequest.sessionEnv`, `request.sessionEnv = os.Environ()` in `run.go`, and the gatecli doc paragraph.** Traces to #157: report the tier env overrides that differ from the session's.
8. **`test_integration_test.go` drops the session's `GOFLAGS`.** Forced by change 7: an inherited `GOFLAGS` would otherwise add an env line to every expected output.
9. **Commit `511b742c`, comment line breaks.** A fixup of the branch's own comments, forced by changes 1–7.

No change traces to no decision.

**Per decision: the changes that build it**

- **#163, verdict from a parsed result:** changes 1 and 3.
- **#163, crash cause in the log:** change 2, only partly (finding 1).
- **#156, the darn prompt names only a runnable form or falls back to `--tier`:** changes 4 and 5.
- **#162, a prompt generated from the declared keys, and an unknown `--gate` refused with the declared keys listed:** changes 4, 5 and 6.
- **#157, env overrides that differ from the session:** changes 7 and 8. The goal offers "report in `lym gate test`" or "list in the prompt"; the diff takes the first.
- **Added after Discussion, the parent's red-first waiver (keep the stencil change):** nothing to build. The diff keeps the stencil change in `contracts/stencils/darn/darn-fragment-make-change-gated.md`, which the waiver keeps. The waiver covers the red-first gate for the whole run, and its rationale names only the stencil file. It licenses no other change, and none relies on it.

**Findings**

1. **LOW, scope.** At `internal/gatecli/tierrun.go:487`, a `vstest` tier whose command holds no `{args}` still logs no crash cause when its test host crashes.
   - The goal says "capture the test host's crash dump or stderr in the gate log when the host crashes", with no condition.
   - The diff adds the results directory and `--blame` only when the command holds `{args}` (the `readsResults && strings.Contains(run.tier.Command, "{args}")` check at `tierrun.go:487`). For any other `vstest` tier, `resultsFileLines` returns nothing and the log carries only the console text, which the issue says gave no cause.
   - `_lym/landing/summary.md:21` and gatecli `doc.go` (diff piece 004, line 100) name only the console-count fallback for such a command, not the missing crash cause.
   - Fix: name the gap as a bound in gatecli `doc.go` and the summary, or capture the cause for those tiers another way, such as reading the host's output from the console text.

I made no tool calls: the inherited context held everything the procedure needed.
