```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals the fixer carried out as proposed (F1–F4), through the fix-proposal lens.

**[BLOCKING] F1: `Aside.Cancel` now keeps a finished cancel, which breaks `lym batten cancel` on a cancelled run**
- Changed text: `internal/runfsm/act.go`, `Judge`: `if aside.Cancel && r.Batten.Cancel != nil && r.Batten.Cancel.Finish == nil`.
- The proposal's premise only considered `resumeAside`. `Aside.Cancel` is also set in `cancelAside` (`internal/battencli/cancelcmd.go:460`), which `cancelVerdict` feeds to `Judge`. `cancelVerdict`'s own doc says "Only a landed run, and a run another machine owns, are refused."
- `internal/battencli/cancel.go:332-338` runs `verdict.Refusal(slug, rerun)` and returns on any non-nil result.
- Before the fix, a cancelled record had its finished cancel dropped by the aside. It read running, so the verdict was act. A second `lym batten cancel <slug> --remove` could then proceed. That is the way forward that `terminalRefusal` names for `Cancelled`: "run `lym batten cancel %s --remove`".
- After the fix, `Derive` reads the run as `Cancelled` and `Act` answers wait/terminal. `cancel.go:336` then returns the terminal refusal, "the run was cancelled; way forward: run `lym batten cancel <slug> --remove`". That refusal sends the user back to the command that just produced it.
- The finished-cancel checks that follow at `cancel.go:339-341` ("already cancelled with its pair kept; run --remove") and the `refuseNothingLeft` path are now unreachable for a finished cancel. The same goes for removing a kept pair via `--remove`.
- Why the tests passed: the fixer's gate runs covered `runfsm`, `battencli`, `tendercli` and `dobbyengine`. No gated test drives the cancel verb over a finished-cancel record through the real `Judge`.
- Sound alternative: make the finished-cancel exemption specific to the resume. Either leave `Aside.Cancel` as it was and have `resumeAside` use a new field (for example `CancelUnfinished`), or have `cancelAside` stop setting `Cancel`.
- The `act_test.go` row `CancelAsideKeepsAFinishedCancel` would then need a matching change, and a battencli cancel test over a finished cancel record should be added.

**F2:** No finding.
- The `Intent == (shedengine.CancelIntent{})` check separates the undecodable case.
- In `readCancelMarker`, an intent from a decoded record always has `Machine` set, because `WriteIntent` stamps it. A zero intent therefore only occurs when the record did not decode.
- `takeCancelMarker` sets `WriterAlive`, so its refusal path is unaffected.

**F3:** No finding. The added resume of `away` reuses the owner set up for the earlier pause check. It runs before the batten-paused section, which uses `pairSlug`.

**F4:** No finding. The `//testtiming:keep` lines sit between the doc comment and the `func` line, matching how other files place the marker. One cosmetic point: the F2 commit contains only `cancelmarker.go`, and the F2 test edit landed in the F4 commit.

Tool calls:
1. `grep` for `Cancel: true` and `Aside{` across non-test code, to find every other user of `Aside.Cancel` (my context did not hold them).
2. `grep` for `cancelAside`, `ReasonTerminal`, `Cancelled` in `cancelcmd.go`, to see how the cancel verb uses the verdict.
3. `grep` for `ReasonTerminal`, `AnswerAct` and similar in `battencli`, to find where a terminal verdict is handled.
4. `sed` of `arm.go:670-690`, to see the run verb's terminal handling.
5. `sed` of `cancelcmd.go:280-300` plus a grep for `Verdict:`, to find who calls `cancelVerdict`.
6. `Read` of `cancel.go:320-360`, to confirm the refusal is returned unconditionally.
