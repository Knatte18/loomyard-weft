```yaml
lens: fix-code-correctness
outcome: reported
```

Task: run the changed code for F1–F4 through the correctness lens. I found one regression, in a caller of `Judge` that the review did not name.

**Findings**

1. **BLOCKING** — `internal/runfsm/act.go:186`. The new `Judge` condition (`aside.Cancel && Cancel != nil && Cancel.Finish == nil`) breaks `lym batten cancel <slug> --remove` over an already cancelled run.
   - `cancelAside` (`internal/battencli/cancelcmd.go:460`) is `Aside{Cancel: true, ...}`, and `cancelVerdict` (`cancelcmd.go:464`) is the real `deps.Verdict` of the cancel verb (`cancelcmd.go:294`).
   - Take a run whose keep-pair cancel finished: `Batten.Cancel.Finish != nil`, `Finish.Remove` false.
   - Before the fix, the cancel verb's aside dropped that intent. `Derive` read the run as running and `Act` answered act.
   - The verb then reached the check at `internal/battencli/cancel.go:340`. That check says "already cancelled with its pair kept; run `lym batten cancel <slug> --remove`" when `--remove` is absent, and otherwise proceeds to finish the removal.
   - After the fix, the finished intent survives the aside, `Derive` reads `Cancelled`, and `Act` answers wait with `ReasonTerminal`.
   - `cancel.go:335` (`verdict.Refusal`) then returns `terminalRefusal`, whose text says "run `lym batten cancel <slug> --remove`".
   - So the verb refuses the command it names, and `--remove` over a kept-pair cancel is no longer reachable. The test cases "FinishedCancelThatKeptItsPairIsFinishedByRemove" and "FinishedCancelThatKeptItsPairRefusesWithoutRemove" use a fake `Verdict`, so they pass anyway.
   - The `act_test.go` row only covers `resumeAside`'s shape.
   - Suggested fix: keep the finished-cancel exemption out of the cancel verb's path. One way is a separate `Aside` field, such as `CancelUnfinished`, that `resumeAside` sets while `cancelAside` keeps dropping any cancel. The other way is to have `cancelVerdict` tolerate `ReasonTerminal` with `State == Cancelled`. Add a test that runs the real `ReadRun` and `Judge` over a finished cancel record.

2. **No finding** — `internal/battencli/cancelmarker.go`, `cancelMarkerRefusal`. `reading.Intent == (shedengine.CancelIntent{})` compiles, since the tests already compare it with `!=`.
   - A decodable unfinished intent always carries `Machine` and `StartedAt`, so it never hits the zero-intent branch.
   - The live-writer branch is checked first. `takeCancelMarker`'s best-effort read, which returns a zero reading on error, also falls through to the live wording. The updated test row matches the new text.

3. **No finding** — the F1 test row "CancelAsideKeepsAFinishedCancel" fails without the fix, because the old code gave Running/Act. The F3 `away` resume assertion and the F4 markers are consistent with the gate runs you reported.

**Calls I made**
- `grep` for `Aside{` and `Cancel: true` across `internal` and `cmd`, to find every caller of `Judge`'s cancel aside.
- `sed` and `grep` on `internal/battencli/cancelcmd.go` for `cancelAside` and `cancelVerdict`, which are not in my context.
- Three greps for how `Verdict`'s result is consumed (`ReasonTerminal`, `Refusal(` and `Verdict()`), to find the consumer.
- `sed` on `internal/battencli/cancel.go:320-350` for the `verdict.Refusal` call and the kept-pair check.
