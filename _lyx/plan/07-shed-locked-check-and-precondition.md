# Card 7 — shed-locked-check-and-precondition

**Create:**
- `internal/shedengine#ReasonPreconditionUnmet`
- `internal/shedengine#ErrLockedCheckRefused`
- `internal/shedengine#TestLockedCheck_RefusesUnderTheRunLock`

**Edit:**
- `internal/shedengine#Shed`
- `internal/shedengine#Shed.Run`
- `internal/shedengine#Shed.Step`
- `internal/shedengine/doc.go`
- `internal/shedverbs#stepCmd`
- `internal/shedverbs#Spec`
- `internal/shedverbs#KindBusy`
- `internal/shedverbs#TestStepCmd_BusyTreatment`

**Uses:**
- `internal/shedengine#newTestShed`
- `internal/shedengine#fixedOutcomeProducer`
- `internal/shedengine#seedStatus`
- `internal/shedengine#commonSeed`

**Intent:**
Give shed the two seams D3 and D11 name, and make a refusal under the run lock reach the Go driver as a hand-off to whoever holds the run, never as a tackler case.

`ReasonPreconditionUnmet` is declared beside `ReasonBounceBudgetExhausted` in `run.go`: the exact prefix of a blocked halt's reason that a producer sets when a precondition the run cannot settle on its own is unmet, matched with `strings.HasPrefix`; shed never writes it itself, and its doc says the reason continues with what is unmet and the way forward (PATTERN-refusal-way-forward).

`Shed` gains the field `LockedCheck func() error`: told, nil meaning no check.
`Shed.Run` and `Shed.Step` call it once they hold the run lock and before `stepLocked` reads or writes the status; a non-nil error is returned wrapped as `fmt.Errorf("%w: %w", ErrLockedCheckRefused, err)`, with the run lock released by the existing defer and the status file untouched.
What it lets a caller do: refuse a drive between taking the run lock and the first status read; it can only stop a drive, never route one.
The input that reaches the refusal is any `LockedCheck` returning an error after a free run lock was taken.
A `LockedCheck` error ends in its own trailing `way forward:` clause (PATTERN-refusal-way-forward), and the `ErrLockedCheckRefused` wrap puts the sentinel's text ahead of the check's, so that clause stays trailing; `LockedCheck`'s field doc says so.

`stepCmd`'s `shed.Step` error branch gains, before its `ErrShedBusy` case, an `errors.Is(err, shedengine.ErrLockedCheckRefused)` case that refuses with `spec.StepBusyKind` and the error's own text, never `spec.StepBusyMessage`, so the check's way forward reaches the caller.
For batten, whose `StepBusyKind` is `shedverbs.KindBusy`, the Go driver's `KindBusy` row exits and leaves the run to its holder, so a step refused because a cancel is ending the run spawns no tackler (`internal/sheddriver/driverules.go`, `decideDrive`'s `KindBusy` case ahead of the tackler rows).
`Spec.StepBusyKind`'s field doc names both sentinels step maps onto it, `shedengine.ErrShedBusy` and `shedengine.ErrLockedCheckRefused`.
`KindBusy`'s doc now reads that busy means the run is held and step leaves it to its holder: the run lock held by a live driver or another `step` invocation, or a check the spec runs before the drive, under the run lock or in its seeding, refusing because the run is held elsewhere or no longer seeded.
`TestStepCmd_BusyTreatment` gains a row: a shed whose `LockedCheck` refuses with a message ending in a `way forward:` clause reports the told `StepBusyKind` with the check's message, not the told busy message, and the reported message ends with the check's `way forward:` clause.

`TestLockedCheck_RefusesUnderTheRunLock` is an untagged test in package `shedengine`, built on `newTestShed`, `fixedOutcomeProducer` and `seedStatus` as `TestRun_LockAlreadyHeld` is: for `Run` and for `Step`, a check that returns an error makes the call return an error matching both `ErrLockedCheckRefused` and the check's error, calls no producer and leaves the status bytes unchanged, and the check observes the run lock held (a `lock.TryAcquireWriteLock` inside it reads not acquired); a nil check drives the producer as before.
`doc.go` names `LockedCheck`, `ErrLockedCheckRefused` and the precondition prefix beside the budget-exhausted paragraph.

**ImpactSummary:** One new nil-default field read only by `Run` and `Step`, and one new case in the step verb reached only by its sentinel; every existing Shed literal leaves the field nil and behaves as before.
