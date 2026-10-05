# Card 5 — compact-cycle

**Edit:**
- `internal/orchengine/watcher.go`
- `internal/orchengine/watcher_test.go`
- `internal/orchengine/state.go`
- `internal/orchengine/prompt.go`
- `internal/orchengine/prompt_test.go`
- `internal/orchengine/doc.go`
- `internal/orchcli/session.go`
- `internal/orchcli/status.go`
- `internal/orchcli/doc.go`
- `contracts/stencils/stencils.go`

**Create:**
- `contracts/stencils/orch/orch-template-compact.md`
- `internal/orchengine/watcher_compact_test.go`

**Uses:**
- `internal/orchengine/config.go`

**Intent:**
Run the compact cycle (D6, D7, D8) when `Config.Mode()` is compact, and keep the clear cycle unchanged for `cycle_mode: clear` (D9).

Stencil: `orch-template-compact.md` is a one-line focus text, with a leading comment in the shape of the other orch stencils, telling the summary to keep runs in flight with slug, state and attach command, open parent-review forks, operator requests and decisions pending, and work half done.
Register it in `contracts/stencils/stencils.go` beside the other orch stencils, and add `RenderCompactFocus(stencilsDir)` to `prompt.go`, rendered single-line through `render` and read from the told stencils directory at call time (PATTERN-stencil-ownership).

Seam: `orchengine.Session` gains `CompactSession(guid, focus string) error`; `internal/orchcli/session.go`'s `runnerSession` delegates to `Runner.CompactSession` with a debug log like its siblings.

State: add `PhaseCompacting` and a persisted `ReadingTurnEnd *shuttleengine.Event` holding the turn end the current reading was taken through, written whenever the watcher stores a reading from a turn end, so a restarted watcher in compacting can re-read the transcript with no turn end in memory.
`ResetForFreshLaunch` clears it.

Watcher, compact mode only:
- `tickIdle`'s trigger selection, gates and D2 re-read are shared with clear mode, except that the soft trigger has no DEFER exchange and that an automatic trigger, hard or soft, is held while `LastDeferral` is within `SoftIdle()`; a requested cycle is never held.
- When a trigger fires: render the focus first (a render failure changes nothing, as for the handoff render), persist compacting via `enter` with `CycleTrigger`, clear the cycle request, call `CompactSession`, then `confirm`.
  Nothing is typed unless the idle probe passed on the same tick.
- `tickCompacting` re-reads the context through `ReadingTurnEnd` every tick.
  The phase completes when the reading is `Compacted` with `BoundaryAt` at or after `PhaseEnteredAt` and the idle probe passes on that tick: the boundary's tokens become the reading, `CycleCount` increments, and the watcher returns to idle.
  An earlier boundary never completes it.
  Past `HandoffTimeout()` it returns to idle with abort reason `compaction timed out`, re-reads the reading, and sets `LastDeferral` to now, which holds the next automatic trigger for `SoftIdle()` (D8).
  Restart: an unconfirmed injection is typed again only when the idle probe passes and no qualifying boundary has been read.
- No handoff file is written and `LastHandoff` is untouched; DEFER handling never runs in compact mode.

Bound: compact mode drops the DEFER handshake, so a background task's in-flight completion has no handshake protecting it if it does not survive `/compact`; `cycle_mode: clear` restores the handshake, and card 12's smoke test asserts a completion survives.
The D8 hold delays the hard trigger by at most `SoftIdle()` per failed compaction.

`watcher_test.go`'s shared env sets `CycleMode` to clear, so the existing four-phase tests run unchanged apart from stencil text, and its fake gains `CompactSession` recording `compact:<focus>`.
`watcher_compact_test.go` covers: a trigger types `/compact` with the rendered focus only on an idle-probe pass;
the phase completes on a boundary at or after entry plus idle, and a boundary before entry does not complete it;
timeout returns to idle with `compaction timed out` and holds the next hard and soft trigger for `soft_idle_s`, while a requested cycle is not held;
a restart in compacting re-types only on idle with no qualifying boundary, and completes without re-typing when one has been read;
a `DEFER` turn end in compact mode changes nothing.
`prompt_test.go` asserts the rendered compact focus is one line.

`status.go`'s long help names the compacting phase.
Docs: `internal/orchengine/doc.go` describes both cycle modes (the four-phase cycle section becomes clear mode, with a compact-mode section beside it), the restart rule, the D8 hold, and updates the residuals: the auto-compaction residual (a threshold above the auto-compaction point now reads correctly, since a boundary is a reading) and the pre-clear-resume residual (applies to clear mode only, since compact mode keeps the session id).
`internal/orchcli/doc.go` names the cycle mode where it describes cycling.
The file header of `watcher.go` describes both machines.

**ImpactSummary:** Adds a second cycle machine to the watcher behind cycle_mode, one Session seam method, one persisted State field and one stencil; clear mode's behaviour and tests stay as they are.
