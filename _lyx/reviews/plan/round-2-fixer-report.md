# Plan-Review round 2 — fixer report

All eight findings were validated against the code and the decision record and fixed.
`lyx loom validate-plan` passes on the fixed plan, with informational findings only.

## Fixed

- **F1** (card 11): the hook-delivered skip now ends the phase at once through `toIdle` with no abort reason, instead of confirming the step through `typeReloadStep` and waiting for a turn end past its own offset.
  Confirmed against `internal/orchengine/watcher.go`: `typeReloadStep` moves `PhaseEventsOffset` to the cursor on each step, and `tickResuming` ends the pointer step only on a turn end read past it, so the operator's turn in the draft case would not count and the phase would end in `resume timed out`.
  The card states why the first turn end after the boundary has already been read by then, and that the stored reading is safe since every idle trigger re-reads a fresh one before firing (`tickIdle`).
  The draft-case row now asserts the phase ends on the skip's tick with no abort reason and no further turn end.
- **F2** (cards 11, 12): `ResumeContext` reads the state through `state.ReadJSON` and returns an error with no mark when the state file is absent, since `LoadState` (`internal/orchengine/state.go`) reads an absent state as phase idle.
  The row is in `resumecontext_test.go`; card 12's CLI test points at it rather than duplicating it.
- **F3** (card 6, overview): card 6 creates `consumeDiscoverSignal`, the stat-and-consume step of the event branch, and an untagged test over it in `t.TempDir()` covering a present file consumed and removed and an absent one answering false.
  The card says `runWatchdogLoop` runs only in the `tmux` tier because it calls `reedengine.ListSessions` against a live tmux server, and raises `internal/reedcli`'s budget by the two new top-level tests.
  The overview's `opener-seam-exported-from-reedengine` decision no longer promises a `reedcli` tier-1 channel test.
- **F4** (cards 7, 8): card 7 lists `internal/shuttleengine/doc.go` and adds the session-start signal, its no-move rule and `RunSessionState.SessionStartAt` to the session-state section; card 8 replaces that section's shadow-mode statement with the reading's two consumers.
- **F5** (card 11): the skip's Bound now names what it lets through, a mark for a dropped render skipping a pointer the session never got, and what bounds it.
- **F6** (card 6): a Bound sentence carries D5's: who can touch the signal and the worst case of one `list-sessions` per 5 s per hub.
- **F7** (card 2): the module-threading test asserts no Warn for a value at or above the floor and the one Warn for a module seeded below it, with no new top-level test.
- **F8** (card 14): the before binary is built from `main`'s tip, extracted with `git archive main` into the scratch directory, since the production `lyx` is built at `prod`.

## Disputed

None.

## Deferred

None.
