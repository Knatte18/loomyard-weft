# Fixer report — plan, round 2

## Changes

- **F1 (BLOCKING), fixed differently in one detail.**
  Card 8's `TestShuttlePairReadings` now drives `shuttlePair.Busy` and `shuttlePair.Background` over a temp run directory: a `lyxcwd.Location` with a temp `HubPath` and `WorktreeName` "p1", a `shuttleengine.Config` whose `RunDir` is a temp directory, and one seeded run directory whose `run.json` reads `outcome` "running", `strandGuid` "g-a" and an `eventsPath` it writes.
  `readTurn` stays the private helper both methods call, reached only through them, and the sentence that a `Busy` reading `Name == SessionBusy` fails the busy/background row is kept.
  Departure from the fix key's "No target list changes": the pair needs an engine whose session-signal reading yields the row's signals, and neither `shuttlefake.Engine` nor any `pausecli` type implements `shuttleengine.SessionSignalParser`.
  The card therefore adds a test-local `signalEngine`, embedding `*shuttlefake.Engine` and answering `ParseSessionSignals` with its scripted signals, to the Create list (`plan:internal/pausecli#signalEngine` and `#signalEngine.ParseSessionSignals`), following card 4's convention of listing its test engines (`pickerEngine`, `pickerOnlyEngine`).
  The third row is named "turn ended" instead of "idle-done": a turn end with no tasks on a run that declares no output folds to idle-stalled (`judgeTurnEnd` in `sessionstate.go`), and the row asserts neither reading either way.
  The test count is unchanged, so the `internal/pausecli` budget raise to 4 stands.
- **F2 (MEDIUM), fixed as proposed.**
  Card 8's interrupt step now runs in three passes over the covered sessions: pass 1 runs steps 1-3 for every session, one settle, pass 2 runs steps 4-5 for the sessions whose first key went, one settle, pass 3 runs step 6 for the sessions whose second key went.
  Each settle sleeps the longest `InterruptSettle()` among the sessions it waits for, since pairs may carry different providers, and is skipped when none.
  The claim stays before each key and an `errEntryGone` at any claim stops the step; every list outcome is unchanged.
  `interruptSession` now takes all the sessions, so its Edit entry carries a re-sign arrow to `func interruptSession(deps Deps, entry pauserecord.Entry, sessions []session, report *PauseReport) (stop bool)`; `Pause` keeps the caller-skip and calls it once.
  The busy-twice `TestPause` row now has sessions `a` and `b`, both busy and pane-running, and asserts interrupts `[g-a, g-b, g-a, g-b]`, `StillBusy == [a, b]` and a clock elapsed of exactly 4 s.
- **F3 (LOW), fixed as proposed.**
  Card 5 lists `TestNewRunner_RefusesUnusableToldPaths` and `TestNewDetachedRunner_RefusesUnusableToldPaths` under Edit, and its Intent bullet now has both pass a lone `{Key: "C-u"}` instead of a lone `{Key: "Escape"}`.
- **F4 (NIT), fixed as proposed.**
  Card 5 now says "by tightening the existing lines below", with no count.

`lyx loom validate-plan` reports `ok: true` with informational `create-new-unit` findings only.

## Disputed

None.

## Deferred

None.
