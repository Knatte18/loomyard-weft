# Card 1 — shuttle-quiet-hold-send-lock

**Edit:**
- `internal/shuttleengine#Spec`
- `internal/shuttleengine#Run`
- `internal/shuttleengine#Run.Send`
- `internal/shuttleengine#Run.sendWithin`
- `internal/shuttleengine#Run.notifyHeld`
- `internal/shuttleengine/hold_test.go`
- `internal/shuttleengine/run_test.go`
- `internal/shuttleengine/doc.go`
- `//cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `internal/shuttleengine#Run.StrandName`

**Intent:** Give shuttle the three small things the seat engine needs, all provider-neutral (D7, D10).
`Spec` gains one knob, `QuietHold bool`, documented as "skips the held-turn parent notice alone": `Run.notifyHeld` returns early when `run.spec.QuietHold` is set, exactly as it does for an interactive run, and nothing else reads the field.
The hold itself, the wait mark, the deadline and attach behaviour are unchanged, which the new `TestWait_HeldTurnEndNotice` row pins: a quiet-hold run's Wait holds, sends no notice, and ends with the same outcome as the loud row.
`Run` gains a `sendMu sync.Mutex` field, the per-run send lock: `Run.Send` and `Run.sendWithin` each take it for the whole idle-wait, typing and confirm sequence, so an in-process notice and the gate's re-prompt or pending send on one run type one after the other, never at once.
The lock only orders sends on one run and skips no idle wait, delivery check or gate evaluation; a send waits on it no longer than the holder's send-ready and submit windows.
`Run.StrandName` returns `run.state.StrandName`, the name reed formed, so a caller can compare it with the name it expected (D4).
A new test in `run_test.go` drives two concurrent `Run.Send` calls, and a `Run.Send` beside a gated Wait's re-prompt send, through the in-package fakes and asserts the pane saw the two lines whole and in sequence, each still preceded by its own idle wait and followed by its delivery check; in the case beside the gated Wait it also asserts the result's `Gate.Attempts` equals that of the same run with no concurrent send, so the gate spends no attempt on the notice.
`doc.go` states the knob and the lock in the hold-notice and verified-send paragraphs.
Raise the `internal/shuttleengine` budget row by the one test this card adds.

**ImpactSummary:** One new Spec field read in one place, one mutex around the two in-process send paths, one accessor; no caller outside shuttleengine changes and every existing send keeps its idle wait and delivery check.
