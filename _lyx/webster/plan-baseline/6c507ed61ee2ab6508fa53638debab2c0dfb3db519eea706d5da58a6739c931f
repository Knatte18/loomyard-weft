# Card 4 — goto-busy-kind

**Create:**
- `internal/shedengine#ShedBusyKind`

**Edit:**
- `internal/shedverbs#gotoCmd`
- `internal/shedverbs#TestGotoCmd_RefusesHeldRunLock`

**Intent:**
Batten's automatic retry (D5) runs the child's goto verb and must tell a goto refused because the driver holds the run lock, which it retries, from any other refusal, which it does not.
It reads the refusal envelope's `kind`, as it already does for `lyx loom start`'s `driver_not_parked` and `driver_busy` refusals, never the message text.

`ShedBusyKind`, declared in `internal/shedengine/errors.go` beside `ErrShedBusy`, is the envelope `kind` of a verb refusal whose error wraps `ErrShedBusy`.
`gotoCmd` writes its refusal with `output.ErrFields(out, msg, map[string]any{"kind": shedengine.ShedBusyKind})` when the `shedengine.Goto` error satisfies `errors.Is(err, shedengine.ErrShedBusy)`, and with `output.Err` as today otherwise.
The message and the exit code are unchanged, so no refusal-spec row changes.

Test: `TestGotoCmd_RefusesHeldRunLock` also asserts the envelope's `kind` is `shedengine.ShedBusyKind`.

**ImpactSummary:** one added envelope field on goto's held-lock refusal; every other goto outcome and every other verb is unchanged.
