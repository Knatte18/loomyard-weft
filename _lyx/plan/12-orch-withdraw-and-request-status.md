# Card 12 — `lyx orch withdraw`, the pending request in `status`, and the pause notice

**Edit:**
- `internal/orchcli#orchCLI`
- `internal/orchcli#Command`
- `internal/orchcli#orchCLI.statusCmd`
- `internal/orchcli#orchCLI.requestCmd`
- `internal/orchcli#orchCLI.refreshCmd`
- `internal/orchcli#orchCLI.distillCmd`
- `internal/orchcli#newTestCLI`
- `internal/orchcli#TestStatus_Envelope`
- `internal/orchcli#TestRefreshAndDistill_RequestTheirOwnMode`
- `internal/orchcli/doc.go`

**Create:**
- `plan:internal/orchcli#orchCLI.withdrawCmd` -> `func (c *orchCLI) withdrawCmd() *cobra.Command`
- `plan:internal/orchcli#orchCLI.hubPauseHolds` -> `func (c *orchCLI) hubPauseHolds() bool`
- `plan:internal/orchcli#cycleRequestField` -> `func cycleRequestField(req orchengine.CycleRequest, pending, held bool) any`
- `plan:internal/orchcli#TestWithdraw` -> `func TestWithdraw(t *testing.T)`

**Uses:**
- `plan:internal/orchengine#WithdrawCycleRequest`
- `internal/orchengine#CycleRequested`
- `internal/orchengine#RequestCycle`
- `internal/orchengine#State`
- `internal/orchcli#orchCLI.requestCycle`

**Intent:**
D5's first, fifth and sixth bullets: under a hub pause the orch deleted `.lyx/orch/cycle-request` by hand; a verb and two fields replace that and say why a request waits.

`lyx orch withdraw` (`withdrawCmd`, audience operator, registered in `Command`) calls `orchengine.WithdrawCycleRequest`.
A pending request of either mode is removed in any phase, and the envelope is `withdrawn: true`.
With none pending it answers `withdrawn: false` with `phase`, the persisted phase, and with `cycle_trigger` only when the phase is neither idle nor resuming, so a caller whose request a cycle start already consumed learns which cycle runs; in idle and resuming the key is absent, since `toIdle` does not reset `CycleTrigger` and `startReload` enters resuming without setting it, so there it can name an earlier cycle.
It never refuses, departing from the board entry's refusal once the cycle started: a refusal would make a caller that lost the race to the cycle start, which it cannot avoid, read as a failed command.
A failed state read or marker removal is the error envelope, a propagated I/O error.
What it can let through, and its bound: it removes only the request marker, never a started cycle's state, and a marker present during a cycle has not started, since a cycle start consumes the marker it starts from under the state lock (card 8), so `withdrawn: true` means no cycle started from it.

`orchCLI` gains `pauseRecordPath string`, set in `Command`'s `PersistentPreRunE` from `hubgeom.PauseGeometry(location).RecordPath`, a failure failing the verb as `watch` does.
`hubPauseHolds` reads the record with `pauserecord.Read` and answers whether `Covering("")` finds a live hub entry, the entry that holds the watcher's cycle machine; an empty path or an unreadable record answers false, as the watcher reads an unreadable record uncovered.

`status` gains `cycle_request`: `cycleRequestField` answers nil when no request is pending, else an object with `mode`, `requested_at` (RFC 3339 UTC, null for an unparseable marker's zero time) and `held_by_pause` from `hubPauseHolds`.
`requestCmd`, shared by `refresh` and `distill`, adds `held_by_pause: true` and `note: "the request waits for the hub pause to be resumed"` when the watcher is live and `hubPauseHolds` answers true; the request is still written.
With no live watcher the request is removed and the envelope is today's.
The `Long` help of `status`, `refresh` and `distill` names the new keys and `lyx orch withdraw`, and `withdraw`'s own `Long` states the answers above.
`refresh`'s and `distill`'s "A request the watcher has not finished within the handoff timeout is abandoned" becomes the rule card 9 built: the timeout counts no time a hub pause held the request.
`internal/orchcli/doc.go` lists `withdraw` among the verbs and states its answers, `cycle_request` and the pause notice.

Tests:
- `newTestCLI` sets `pauseRecordPath` to a path under the test's temp directory, absent until a test writes it with `pauserecord.Update`.
- `TestWithdraw` is new: a pending clear request answers `withdrawn: true` and leaves no marker; no request in phase idle answers `withdrawn: false, phase: "idle"` with no `cycle_trigger` even though the state records `CycleTrigger: "requested"`; no request in phase resuming with a recorded trigger answers no `cycle_trigger`; no request in phase handoff-requested with trigger requested answers `cycle_trigger: "requested"`; a request written during a running requested cycle (phase clearing) is withdrawn with `withdrawn: true`.
- `TestStatus_Envelope` gains rows: no marker gives `cycle_request: null`; a compact request gives its mode, time and `held_by_pause: false`; the same under a live hub pause entry gives `held_by_pause: true`; the same with the pause record holding unreadable bytes gives `held_by_pause: false`; an unparseable marker gives an object with an empty `mode` and `requested_at: null`.
- `TestRefreshAndDistill_RequestTheirOwnMode`'s live-watcher row gains a variant under a live hub pause entry answering `held_by_pause: true` and the note, with the request still pending; its no-watcher row keeps no `held_by_pause` key even under a pause.

**ImpactSummary:** One new operator verb and three envelope additions inside orchcli over orchengine's marker functions; no other module reads these envelopes, and the creel refusal that names `distill` and `refresh` stays true.

**Commit:** 12: lyx orch withdraw, the pending request in status, and the pause notice
