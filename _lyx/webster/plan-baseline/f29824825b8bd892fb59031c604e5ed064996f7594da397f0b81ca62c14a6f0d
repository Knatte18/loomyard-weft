# Card 11 — watcher-reloads-fresh-compaction-only

**Edit:**
- `internal/orchengine/watcher.go`
- `internal/orchengine/watcher_test.go`
- `internal/orchengine/doc.go`
- `internal/orchcli/session.go`

**Uses:**
- `plan:internal/shuttleengine#CompactionBoundary`

**Intent:** D8, watcher half: an auto-compaction boundary enters the reload sequence only when exactly one turn end follows it in the transcript and that turn end is the one being read, the newest turn end among the events read on the tick.
A reload typed after turns that ran without it no longer answers the compaction it was meant for, and a watcher restarted on an old cursor otherwise types a reload into a session working normally (#390).

`orchengine.Session.CompactedSince` returns `(shuttleengine.CompactionBoundary, bool, error)`, and `runnerSession.CompactedSince` passes the Runner's answer through unchanged.

In `Watcher.tick`, where the idle phase asks for a boundary through the newest turn end read (`lastTurnEnd`):

- `TurnEndsAfter > 1` (two turn ends read in one tick, or a restarted watcher on an old cursor finding an old boundary): move `State.CompactionBaseline` to the boundary's `At`, save, log at Info that a stale compaction boundary was passed without a reload (strand, boundary time, turn-end count), and type nothing.
- `TurnEndsAfter == 1` and `ReadTurnEndAfter`: hold the boundary in `w.compactedAt` as today, so `tickIdle` enters the reload sequence once the idle probe passes.
- Otherwise (one later turn end the watcher has not read yet, or none): type nothing and leave the baseline, so a later turn end evaluates the boundary again.

A tick that read a turn end sets `w.compactedAt` from its own evaluation alone: to the boundary's `At` under the second rule, and to zero under the first and third rules and when no boundary is found.
A boundary an earlier tick held while its idle probe failed therefore never outlives the evaluation that finds it stale, and `tickIdle`, which runs after the evaluation in the same tick, enters the reload only for a boundary the current evaluation confirmed.
A tick that read no turn end leaves the held boundary as it is, except that `initCursor` clears it when the watcher binds to another strand, so a boundary held for the old strand never reloads the new one.

Rewrite the "After an auto-compaction" bullet of "The reload sequence" in `doc.go` with these three rules and the held-boundary rule.

Bound (R3): a compaction mid-turn whose turn end is followed by another before the watcher reads gets no reload; the role pointer reaches the session at its next cycle.

Tests in `watcher_test.go`, through `Watcher.Tick` over the `fakeSession`, whose boundary answer becomes a `CompactionBoundary` per turn-end message: a watcher started on an old events cursor whose transcript shows a compaction followed by two turn ends types nothing and moves its baseline to the boundary; a boundary with one later turn end not yet read (`ReadTurnEndAfter` false) types nothing and leaves the baseline, then the tick that reads that turn end enters the reload; a boundary with no later turn end types nothing and leaves the baseline; a boundary held by a tick whose idle probe failed, followed by a tick that reads a second turn end, types nothing, moves the baseline and holds nothing; a boundary held by a tick whose idle probe failed, followed by a strand change and a tick that reads no turn end, types nothing.
`TestWatcher_AutoCompactionReloadsSkillsRoleAndPointer`, `TestWatcher_AutoCompactionBoundaryAtOrBeforeBaselineTriggersNothing` and `TestWatcher_AutoCompactionWaitsForIdleProbe` move to the one-turn-end, read boundary.

**ImpactSummary:** Changes the orchengine Session seam and the watcher's auto-compaction check; only the orch adapter and the watcher test fake implement the seam.
