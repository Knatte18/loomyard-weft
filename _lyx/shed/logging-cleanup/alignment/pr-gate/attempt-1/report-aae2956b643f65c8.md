---
verdict: APPROVED
---

## Scope

PR 604 at `25bcbc9b09e47c87c8de3697b01f050c688697d0`, diffed against `main` from merge base `30385e1ae`, which is `main`'s head, follows the board entry's goal and the decision record.
I found no scope deviation.

- reed-resize-pane-count: `internal/reedcli/watchdog.go` retires the watchdog when its binary is replaced (D2).
  It records the executable's identity and modification time at start and re-checks them on the existing timer tick, with no new wakeup.
  A failed stat counts as unchanged and logs at Debug.
  On a change it releases the lock, respawns through `reedengine.SpawnWatchdog` with `testing.Testing()` as the suppress value, logs at Info and exits 0.
  The resize re-apply failure goes through a per-watcher `logger.Episode` (D3).
  Lock contention logs at Debug (D4).
  The help text names D18's one-time manual step.
- log-level-hygiene: `reportStuck` logs at Warn for failed steps and the new `reportWait` logs at Info for waits.
  Waits are deduped on a caller-passed key while the reason file still gets the full text (D5).
  The halt line is Info (D6) and the session-state-at-notice line is Debug (D7).
  The orch treats busy/background and asking/not-idle as agreement and logs at Info once per episode keyed on `Since` (D8).
  A draft hold logs the constant reason "the input box holds a draft" (D9).
  `SessionNameDrift` logs a missing transcript at Debug and other failures at Warn once per session id; the create-worktree line drops its mutations field (D10).
  The D11 audit sites follow the plan's verdict table in `00-overview.md`, with guards in reed, battenshed, shuttle, orch, shedverbs, loomcli and landingshed.
- Policy and PATTERN (D12): `logger.go` gains the three rules, and `PATTERN-log-levels` sits under `## Daemons and waits` with no `(test)` tag.
  The same commit shortens four other PATTERN lines.
  Card 2 calls for this because of the 15564-byte `lint-pattern` headroom, and the detail stays in each line's background file.
- Test noise (D13): `logger.DefaultOutput()` returns `io.Discard` under `testing.Testing()`.
  Tests that restore the stderr half now restore that value.
- run-log-archive (D14, D15): `pairteardown.archivePairLogs` runs after the run-lock wait and right before `remove`.
  It writes under `fabricengine.LogArchiveDir`, which is `<hub>/_board/.lyx/log-archive`.
  Entry directories are created exclusively with a numeric suffix on collision, and a failed copy removes its partial entry and logs at Warn.
  The prune applies a 30-day constant, skips names that don't parse and plain files, and treats an already-gone path as removed.
  `PATTERN-pair-teardown.md` and the `doc.go` file are updated.
- batten-land-notice (D16, D17): the Land row always queues `<slug>: landed`, adding `; newly ready: …` when tasks became ready.
  The `ready` marker literal is kept.
  The orch stencil routes every `landed` notice to After landing and adds the next-wake board check, the notice-log fallback and the deploy-recovery check.
  `claims_test.go` and `docs/overview.md` are updated.
- Merge-Review carry-over `restep-and-verify-retry-warn-every-poll` is addressed by `22aa94bfc` and `25bcbc9b0`.
  The binary re-step seam reads and the Go verify-retry read now log once per episode, and the dirty-pair skip logs at Info only when the path list changes.
- The other open risks are accepted in the record and don't block:
  - Windows is unverified for D2.
  - Verify-log sizes were never measured.
  - D18 depends on the operator.
  - D11 may have missed sites, which the parent bounded on 2026-10-13.

## Overlaps and landing order

- Peer `shuttle-gate-early-done` (row 9, own row 22): both runs change `internal/shuttleengine` and `claudeengine`.
  This run adds a draft kind to the idle probe and to `busyReading`/`ErrSessionBusy` in `send.go` (D9), and changes how the session-name read failure is logged.
  The peer changes when `turnStartHold.holds` releases a hold: an in-flight transcript reading vetoes the idle-pane override.
  It also adds a new optional engine capability.
  These are different behaviours with no contradicting decision.
  The peer keeps the hold's release Warn, which this run does not touch.
  The peer has no changed paths yet.
  Landing order: this run is furthest along (row 22 against 9), so it lands first and `shuttle-gate-early-done` takes its changes at Merge-In.
- Parent commits: `lyx loom peers` lists none, and `main` sits at the merge base.
- Board note `stale-binary-daemons` (Ready, not a running peer) covers restarting the reed watchdog, status panes and `lyx orch watch` on a new binary.
  This run's D2 handles only the reed watchdog and records the other processes as out of scope.
  The note is therefore not made obsolete, and its design should build on D2's retire-and-respawn.
- Shared bookkeeping, no escalation: `cmd/lyx/testdata/test-budget.yaml`, `docs/overview.md`'s Land line, and the new `PATTERN-log-levels` index line.
