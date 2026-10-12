# Lens review

Every lens fork returned a report.

## criteria-coverage

- MEDIUM, D9 hand-back and spawn settle records untested — fixed: every `TestGoDriver_Settle` hand-back row asserts the `hand-back` record with settler `parent` (card 12), and every agent row asserts its `spawn` record (card 13).
- MEDIUM, D11 pause untested for the verify goto and the arbiter spawn — fixed: card 12 adds a pause row before `Goto`, and card 13's pause row holds the arbiter too.
- MEDIUM, D3 relay of `parent_notice` on a refused arbiter spawn unreached — fixed: card 13's `arbitrate` relays it on a refusal, with a row.
- LOW, D7 session end ends a live tackler — fixed: card 15's `doc.go` Tackler section states it; no code changes, since the tackler is a strand in the pair's reed session.
- LOW, D7 stop-report line on a moved run unasserted — fixed in card 13's row.
- LOW, D2 handshake switch untested and reaping on the wrong card — fixed: card 16's `startCmd` passes `c.driverLockHeld` and `TestRunDriverSpawnAndWait_GoArm` uses it over the real lock; card 15 prescribes the reaping.
- LOW, D1 wiring calls untested — rejected: `ResolveDriverKind`, `resolveSeedDriver` and `childDriverFrom` are pinned, and the remaining calls are one-line wirings whose only Tier 1 reach would need a hub fixture.
- LOW, `PATTERN-friction-capture` unstated — fixed: card 12 states the halt note comes from each step child's `loomAfterStep`, which `halt_test.go` already pins.
- LOW, batten-side "run_settings after seeding" — rejected: Seed-Child never re-runs a Done row (`internal/battenshed/deps.go`, `ErrDisagreeingChildSeed`'s comment), so later `run_settings` never reach a seeded child.

## row-reachability

- BLOCKING, `resumeOpenAttempt` reuses a verdict already handed back — fixed, since `arbiterCall.settleExisting` (`internal/loomcli/arbiter.go:193-220`) and card 7 reuse any attempt for the key.
  Card 13 now treats an attempt as open only when no later `hand-back` settle record exists for its key.
  Card 7's `settle` skips such a closed attempt through a `Closed` dep.
  Each gets a row.
  The arbiter's shared reuse is left unchanged as a stated bound.
- MEDIUM, the loop-level reuse row cannot fail — fixed: the row now asserts `Tackler` is called before any `Step`, and reuse stays `TestTacklerCall_Rerun`'s.
- LOW, driver-lock rows need the real lock — fixed: card 16's rows take the real `shedrun.DriverLock`, as the `runLock` column does.
- LOW, card 20's rows could wait forever — fixed: each Call ends through a scripted `running` status.

## forced-edits

- BLOCKING, the tmux smoke tests of the go arm are on no card — fixed: card 16 lists `seedGoDriverRun`, `findDriverPIDs`, `waitRunLockFree`, `TestSmokeRunStandalone`, `TestSmokeBootstrapLifecycle` and `TestSmokeStatusStrandAcrossDriverSeeds`, with the tackler and arbiter turned off in the fixture.
  The `TestIntegrationDriverBootstrap_ReturnsWithoutWaitingOnTheDriver` part is rejected: that test drives the llm arm's `startLLMDriverArm` directly (`internal/loomcli/integration_driverbootstrap_test.go:20`).
  `TestSmokeStepBootstrapWiring` drives `lyx loom step`, not start (`smoke_bootstrapwiring_test.go:71`).
- MEDIUM, `docs/overview.md:591` says `--child-driver` defaults to `llm` — fixed on card 18.
- MEDIUM, `BattenDir` does not exist — rejected: `internal/battencli/wire.go:708` and `arm.go:679` call `BattenDir(location, slug)`, declared in `internal/battencli/paths.go`.
- LOW, stale handshake comments in `bootstrap.go` and the refusal text — fixed: card 16 lists `awaitRunLock` and `dispositionForHandshake` and rewords the refusal.
- LOW, `driverWatched` doc and `internal/shedrun/doc.go` — fixed on card 11.
- LOW, `NewSeedChild` doc and the `seamchild.go` header — fixed on card 18.

## test-economy

- LOW, card 16's held-driver-lock rows belong in `TestRunDriverSpawnAndWait_LiveDriverWithoutParkMarker` — fixed: a liveness column there; the new test keeps only the spawn rows.
- LOW, the awaiting-with-decision row repeats the spawn row — fixed: dropped.
- LOW, `TestTacklerTampered` is a per-helper test — fixed: `tacklerTampered` is dropped; the cases are `WorktreeSnapshot.Equal` rows in card 3's `TestTacklerRecords`.
- LOW, three top-level loop tests drive one entry — fixed: cards 13 and 14 add rows to `TestGoDriver_Settle`.
- LOW, loop rows repeat notice-selection rows — fixed: the loop rows assert only that `Notify` was reached.
- LOW, `TestSyncWatchedMarker` repeats the format round-trip — fixed: it asserts the bytes written.
