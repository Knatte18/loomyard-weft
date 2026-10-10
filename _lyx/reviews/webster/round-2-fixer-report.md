# Fixer report — webster round 2

The review's verdict is APPROVED with no findings, so nothing was fixed and no commit was made.

Validation: I checked the review's three focus verdicts against the code in the range (`git diff $(git merge-base main HEAD)..HEAD`) and agree with each.

- `BootSignalWakesBackedOffDiscovery`: after the prime's entry, unsignalled cycles fall 1, 3, 7, 15, 31, 63 and 127 bases later (`discoveryWake.afterCycle` through `reedengine.NextWakeCadence`).
  The pair boots past the 63rd and the wait ends at `primeEnteredAt + 120 * base`, so only the signal branch of `runWatchdogLoop` can enter it in time.
- `probeIdle` clears `lastHeld` and `heldSince` on every passing probe, so a later hold with the same reason logs again whatever was typed in between.
- `probeIdle` clears `State.Stuck` on the persisted `StuckByHold`, not on watcher memory, so a restart that runs `initCursor` does not strand a hold's reason on `lyx orch status`.

## Disputed

None.

## Deferred

None.
