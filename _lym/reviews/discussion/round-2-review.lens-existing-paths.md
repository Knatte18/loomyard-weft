```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths, for each state a decision changes, every existing path that writes, resets or reads it.

**Paths traced, per decision**

- **D1, Stop output without a block.** All of these survive:
  - `creelStopHookCommand` (`settings.go:146`) still reads hook-read's exit 0 as "no turn end recorded". `hookReadExit` (`hookread.go:72`) still exits 0 exactly when the output is written.
  - Claude 2.1.312's Stop runner pushes additional context into the continuation list (`He.push` unless `Ugt`). It then continues the turn with `stopHookActive:!0` and `stopHookBlockingCount:Me`, the same as a block. So `ClaimAtBoundary`'s reset on `!StopHookActive` (`verbs.go:324`) and `stopBlockCap` still count right.
  - Idle detection still reads the events file's Stop record. No record is written for a continued Stop.
- **D2, PostToolUse output.** Survives. `creelMailHookCommand` passes the output through, and the exit code does not change.
- **D3, ring claim reads `hookWakeState` / `unreadLocked` (`pool.go:446`).** Survives. These exclude reserved ids, so while the ring's claim is reserved, `wakeHookStrand` (`deliver.go:259`) sees either an empty set or only the over-budget remainder. With no waiter registered and the ring unanswered, the 30 s window governs, as D3 states.
- **D3, `markRingAnswered`** (a probe that reads a turn in progress, `deliver.go:275`). Survives. It can mark the ring answered before the ack, but the released mail is then taken by that turn's Stop claim.
- **D3, `ClaimAtBoundary` marking the ring answered at hand-out (`verbs.go:351`).** Unchanged for boundary claims and does not conflict with the ring's own claim. Survives.
- **D3, `resetStopBlocks` after a ring (`verbs.go:448`).** Survives. The Stop count still starts over, and the woken turn's `stopHookActive:!0` does not matter because the count was reset.
- **D3, `HookWait` supersede via `answerLocked` (`verbs.go:410`, `pool.go:377`).** Survives. A waiter that was already rung has left the map, so supersede cannot reach its claim.
- **D3, `awaitOutcome`'s close and shutdown arms (`server.go:388`).** Would break, but the record covers it: D3 releases on every end before the journaled ack, and the Notes name the race.
- **D3, daemon shutdown on binary replace (`server.go:161`) and restart.** Survives. The `reserved` map is lost, the mail stays unread, and the old client's resumed request is unmarked, so it gets a ring without mail.
- **D3, flag sync (`flags.go:17`, `:30`).** Survives. It runs under the store's lock, and the ring's ack journals the read through the same `Reading.Ack` path.
- **D3, `Client.CallWatching` reconnect-and-resume for `Wait` requests (`client.go` `CallWatching`, `resume` at `client.go:176`).** Breaks; see F1.
- **D3, Claude 2.1.312 asyncRewake on exit 0.** Survives. Stderr is used only on `code===2`, so exit 0 drops it, as D3 states.
- **D4, other readers of the waiter entry.** Survives. Only `buildSettings` (`settings.go:403`) writes it. A pre-change session runs the new `lym` binary's `hook-wait` with old settings, so its wake carries the default labels; D4's Bound states this.
- **D5, logging.** Survives:
  - The daemon logs through `internal/logger`, as in `server.go:471`.
  - Mail that cannot be claimed (over `hookFieldCap`, or held by another sender) keeps its flag standing. That gives one `Debug` claim line per tool call, which is the case D5 names.
  - The bell's `typeBell` stays the single send site.
- **Other `OutcomeRing` / `RewakeOutput` / `BoundaryBlock` readers outside creel.** None in production; the grep found only `hookwait.go`, `hookread.go`, the seam and claudeengine. Survives.

**Findings**

- **F1 — LOW, design, D3 / Notes for the plan writer.** The existing Go client resumes a waiting call after a lost acknowledgement, which undoes D3's "never resumes the wait". The record names only the client's auto-ack half of that path.
  - Evidence: `internal/creelengine/client.go` `exchange` returns `&connLostError` when the ack write fails or `awaitAckConfirmation` reads a closed connection. `CallWatching` then reconnects and calls `resume` whenever `req.Wait` is set, and `resume` resends `VerbHookWait` unchanged (`client.go:176`). `internal/creelengine/doc.go:140` states "A dropped hook-wait is resumed by sending the same request again".
  - Effect: with D3's handler in place, `hook-wait` writes the lines to stderr and the ack connection drops. The client then waits again instead of returning, so stderr holds stale lines that a later exit 2 would deliver. That is the duplicate D3 and the support log's rejected "Resume the wait after a failed acknowledgement" rule out.
  - Why LOW: D3 states the required behaviour and an acceptance criterion tests it. The record is silent only on the existing path that would defeat it.
  - Fix: name the client's resume-after-ack in D3 or the Notes beside the auto-ack note, and the `doc.go` resume sentence among the docs Scope already lists.

**Tool calls**

1. Grepped for `OutcomeRing`, `VerbHookWait`, `RewakeOutput` and `BoundaryBlock(` across the repo, and listed `flags.go`'s functions. The context did not show the readers outside creelengine or how flags sync.
