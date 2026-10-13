# Card 1 — shuttle-turn-in-flight-veto

**Create:**
- `plan:internal/shuttleengine#TurnProgress` -> `type TurnProgress int`
- `plan:internal/shuttleengine#TurnProgressUnknown` -> `const TurnProgressUnknown TurnProgress = 0`
- `plan:internal/shuttleengine#TurnInFlight` -> `const TurnInFlight TurnProgress = 1`
- `plan:internal/shuttleengine#TurnOver` -> `const TurnOver TurnProgress = 2`
- `plan:internal/shuttleengine#TurnProgressReader` -> `type TurnProgressReader interface`
- `plan:internal/shuttleengine#sessionFakeEngine.TurnStartProgress` -> `func (e *sessionFakeEngine) TurnStartProgress(SessionSignal) TurnProgress`
- `plan:internal/shuttleengine#clearableBoxEngine.TurnStartProgress` -> `func (e *clearableBoxEngine) TurnStartProgress(SessionSignal) TurnProgress`

**Edit:**
- `internal/shuttleengine#unmatchedTurnStart` -> `func unmatchedTurnStart(engine Engine, eventsPath string) (turnStart SessionSignal, restarted, found bool)`
- `internal/shuttleengine#turnStartHold.holds` -> `func (h *turnStartHold) holds(engine Engine, turnStart SessionSignal, restarted, paneIdle bool, now time.Time) bool`
- `internal/shuttleengine#busyReading`
- `internal/shuttleengine#sessionReadiness`
- `internal/shuttleengine#Run.boundaryIdle`
- `internal/shuttleengine#Run.startHoldReleased`
- `internal/shuttleengine#awaitIdleSession`
- `internal/shuttleengine#sessionFakeEngine`
- `internal/shuttleengine#clearableBoxEngine`
- `internal/shuttleengine#clearableBoxEngine.ParseSessionSignals`
- `internal/shuttleengine#TestSend_WaitsForIdleSession`
- `internal/shuttleengine#TestGate_WaitsForIdleWriter`
- `internal/shuttleengine#TestRunner_SessionIdle_ReturnsScriptedClassification`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/wait.go`

**Intent:**
D1, and the shuttleengine half of D2: an unmatched turn start that the engine reads as still in flight vetoes the 10 s idle-pane override (`turnStartIdleOverride`), so neither the gate boundary nor a verified send nor `Runner.SessionIdle` takes a running turn as over.

In `sessionsignal.go`, beside `SessionProber`, declare `TurnProgress` with its three values in one `const` block over `iota`, in this order: `TurnProgressUnknown` (the zero value), `TurnInFlight`, `TurnOver`.
Declare the optional capability `TurnProgressReader` with one method, `TurnStartProgress(turnStart SessionSignal) TurnProgress`: it reports whether the turn `turnStart` began is in flight, over, or unknown to the engine, and any failure to read reads unknown.
Its doc says an Engine without it leaves every turn start unknown, which keeps the idle-pane override as before.
`shuttleengine` never sees a transcript shape; only these three values cross the seam (`PATTERN-shuttle-provider-seam`).

`unmatchedTurnStart` also reports `restarted`: true when a `SessionSignalSessionStart` of any source lies after the returned turn start in the events file.
In its loop, a turn start sets `restarted` false, a session start sets it true while a turn start is pending, and a turn end or API-error turn end clears `found` as today.

`turnStartHold.holds` takes `restarted` and decides in this order:
a turn start once released stays released (unchanged);
a reported interrupt from `SessionProber.TurnStartInterrupt` releases at once (unchanged, still first);
then, when `restarted` is false and the engine implements `TurnProgressReader` and reads `TurnInFlight`, the hold resets `idleSince` to zero and holds, whatever `paneIdle` says;
otherwise the existing pane-idle timer and `turnStartIdleOverride` decide, with the existing release Warn as the only log line.
The veto logs nothing per poll.
`TurnOver` and `TurnProgressUnknown` both fall to the existing timer, so an engine without the capability, a turn start whose payload names no transcript, and an unreadable transcript keep today's override.

Bound: the veto only lengthens a hold; it never lets a send, a gate evaluation or an idle report through earlier.
A turn read in flight forever holds until a turn end, a reported interrupt, a later session start, the verified send's `send_ready_timeout_s` (`ErrSessionBusy`, retried on later ticks without spending the gate's budget) or the run deadline, whose classification is unchanged; no negative-outcome path changes (`PATTERN-completion-signal`).

The four callers take the new results: `busyReading` and `sessionReadiness` pass `restarted` to `holds`, `Run.startHoldReleased` does too, and `Run.boundaryIdle` reads only `turnStart` and `found`.
Update the comments that describe the release: `awaitIdleSession`'s doc, `sessionReadiness`'s doc, `holds`' doc, the sentence of `wait.go`'s file comment that says the boundary is restored when the engine reports that turn interrupted or the pane has read idle for turnStartIdleOverride (that sentence only; `wait.go` has no code change on this card), and `doc.go`'s paragraph on the unmatched turn start (the line beginning "An unmatched turn start releases after the pane has read idle"), which gains the in-flight veto, its session-start exception and that `Runner.SessionIdle` reports busy while a turn reads in flight.
Both comment sites stay one sentence per line.

Fakes: `sessionFakeEngine` and `clearableBoxEngine` each gain a `progress TurnProgress` field and a `TurnStartProgress` method answering it for any turn start, with a `var _ TurnProgressReader` assertion beside each fake's existing assertions; their doc comments name the new answer.
The zero value is `TurnProgressUnknown`, so every existing row keeps its behaviour.

Tests, all on fake clocks, as rows of existing tests:
- `TestSend_WaitsForIdleSession` gains a `progress TurnProgress` row field, set on `idle.progress`, and these rows over `events: "START\n"` with the pane idle from the start:
  - "an in-flight turn start holds an idle pane past the override" — `reading: true`, `progress: TurnInFlight`, `wantBusy: []string{"a turn is running"}`, `wantReason: "a turn is running"`: the send waits out the 60 s send-ready window and fails `ErrSessionBusy`, well past `turnStartIdleOverride`.
  - "an in-flight turn start under an unknown reading holds past the override" — `progress: TurnInFlight`, no `reading`, `wantBusy: []string{"has no turn end"}`: the pane-reading path through `busyReading` holds too.
  - "a reported interrupt releases an in-flight turn start at once" — `reading: true`, `interrupted: true`, `progress: TurnInFlight`, typed at 0 with no Warn.
  - "an over reading still releases by the idle override" — `reading: true`, `progress: TurnOver`, `wantTypedMin: turnStartIdleOverride`, `wantTypedMax: 15 * time.Second`, `wantWarns: 1`.
  - "a session start after an in-flight turn start releases by the idle override", over `events: "START\nSESSIONSTART:resume\n"`, `progress: TurnInFlight`, `reading: true`, `wantTypedMin: turnStartIdleOverride`, `wantTypedMax: 15 * time.Second`, `wantWarns: 1`, through `sessionReadiness`; `sessionFakeEngine.ParseSessionSignals` already reads `SESSIONSTART:<source>`.
  - "a session start after an in-flight turn start under an unknown reading releases by the idle override": the same with no `reading`, through `busyReading`'s pane path.
    One source stands for all three, since `unmatchedTurnStart` never reads a session start's source.
  The rows with no `reading` leave liveness unproven, so the session state is unknown and the pane decides, as the existing "under an unknown reading" row does.
- `TestGate_WaitsForIdleWriter` gains a `progress TurnProgress` field set on `rig.engine.progress`, and the row "an in-flight turn start holds the boundary past the override until its turn end": `progress: TurnInFlight`, `turnEndAt: 14 * time.Second`, `wantTypedMin: 14 * time.Second`, `wantTypedMax: 16 * time.Second`.
  The rig's events `"STOP:turn1\nSTART\n"` are the bench incident shape: a turn end, then a turn start that no turn end matches.
  Without the veto the boundary is restored at about 10 s and the re-prompt types before 14 s, so the row fails.
  A second row, "a session start after an in-flight turn start restores the boundary by the override", seeds the rig with `"STOP:turn1\nSTART\nSESSIONSTART:resume\n"`, `progress: TurnInFlight` and no turn end, and wants the re-prompt typed within `[turnStartIdleOverride, 15 * time.Second]`, so `Run.startHoldReleased` is pinned to pass `restarted`.
  For it, `clearableBoxEngine.ParseSessionSignals` also reads a `SESSIONSTART` or `SESSIONSTART:<source>` line as a `SessionSignalSessionStart` with that source, and its doc comment says so.
- `TestRunner_SessionIdle_ReturnsScriptedClassification` gains a `progress TurnProgress` engine field set on `engine.progress`, and the row "an in-flight turn start stays held past the idle override": probes `{events: "START\n", capture: idleFrame, want: IdleProbe{Reason: "a turn is running"}, wantReason: "a turn is running"}` then `{advance: turnStartIdleOverride, capture: idleFrame, want: IdleProbe{Reason: "a turn is running"}, wantReason: "a turn is running"}`.

**ImpactSummary:** two re-signed package-private members whose four callers all sit in `send.go` and `readiness.go`; one new optional interface no existing engine or fake must implement; no exported signature changes.

**Commit:** 1: shuttle: an in-flight turn vetoes the idle-pane override of an unmatched turn start
