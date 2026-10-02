# Card 9 — soft-trigger-watcher

**Edit:**
- `internal/orchengine/state.go`
- `internal/orchengine/watcher.go`
- `internal/orchengine/watcher_test.go`
- `internal/orchengine/state_test.go`
- `internal/orchengine/doc.go`

**Uses:**
- `plan:internal/orchengine#Config.SoftThreshold`
- `plan:internal/orchengine#Config.SoftIdle`
- `plan:internal/orchengine#RenderSoftHandoffInstruction`

**Intent:**
Add the soft trigger to the watcher, keeping `threshold_tokens` as the hard cap and the four-phase cycle as it is.

State gains `CycleTrigger string` (`soft`, `hard` or `requested`, the trigger that started the current or last cycle, with exported constants for the three values) and `LastDeferral time.Time` (when the last `DEFER` turn end was read; zero when none).
Both are persisted, so a restarted watcher knows which stencil its unconfirmed handoff request re-sends and whether a deferral is still pending.
`ResetForFreshLaunch` keeps both.

`tickIdle` picks the trigger in a fixed order:
a known reading at or over `Threshold()` is `hard`, whatever `idle_grace_s` and `soft_idle_s` are;
else a pending cycle request is `requested`;
else a known reading at or over `SoftThreshold()` and below `Threshold()` is `soft`, so a soft threshold at or above the hard one never fires.
`hard` and `requested` keep today's gates: the newest event is a turn end read at least `IdleGrace()` ago, and the idle probe passes.
`soft` requires the newest event to be a turn end (`EventStop` or `EventWaiting`) read at least `SoftIdle()` ago, no pending deferral (`LastDeferral` is zero or at least `SoftIdle()` before now), and the idle probe passing.
The trigger is recorded on State before the phase is entered, and `soft` sends `RenderSoftHandoffInstruction` where the others send `RenderHandoffInstruction`.
`tickHandoff`'s re-send of an unconfirmed request renders the stencil matching `CycleTrigger`.

`DEFER` in a soft cycle: a turn end read in the handoff-requested phase carries `DEFER` when its `Message`, trimmed of whitespace, equals `DEFER` exactly;
record the first such turn end and its read time in `phaseEvents`.
When one is seen and the handoff file is not seen written, re-stat the file on that tick;
if it is still not written, set `LastDeferral` to that read time and return to idle with `LastAbortReason` `deferred`.
A written file wins over `DEFER`, and the cycle proceeds through today's clear gate.
In a `hard` or `requested` cycle `DEFER` is never recorded, and the phase waits for the file until the handoff timeout as today.
The soft trigger only adds an earlier, deferrable entry into the existing handoff-first cycle, so it never reaches `/clear` without a written handoff, and nothing is typed unless the idle probe passed on the same tick.

Watcher tests with the fake session:
the soft trigger fires on an `EventStop` and on an `EventWaiting` turn end once `soft_idle_s` has passed, and not one tick before;
a `DEFER` reply with no handoff file returns to idle with `deferred` and blocks the next soft attempt until `soft_idle_s` after the reply was read, then allows it on a fresh turn end;
a reply containing `DEFER` among other text does not defer;
`DEFER` with the handoff file written proceeds to clearing;
`DEFER` in a hard cycle and in a requested cycle is ignored and the phase aborts only at the handoff timeout;
the hard trigger still fires on `EventWaiting` after the idle grace;
a reading at or over `threshold_tokens` starts a hard cycle when `idle_grace_s` exceeds `soft_idle_s`;
a soft threshold at or above the hard one never fires;
`CycleTrigger` reads `soft`, `hard` or `requested` accordingly;
a watcher restarted in a soft handoff-requested phase re-sends the soft stencil.
State test: `CycleTrigger` and `LastDeferral` round-trip through save and load, and `ResetForFreshLaunch` keeps them.

In the orchengine package doc, rewrite "Idle rules" to describe both triggers and their gates, add the `DEFER` rules to the four-phase cycle section, and name `CycleTrigger` and `LastDeferral`.

**ImpactSummary:** Changes the idle-phase trigger decision and the handoff phase's abort rules inside orchengine's watcher; the clearing and resuming phases and every hard-cap path keep their behaviour.
