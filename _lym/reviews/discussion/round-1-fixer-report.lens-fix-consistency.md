```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fixer's changes to the decision record still agree with the sections around them (fix-consistency lens).

1. LOW — D6's new text "the key is kept in the run's watch entry of the state record (D4)" against D4's "Each run's entry holds the last reading the watch handled, the copied land_notice reading (D3) and the run's owed watch lines".
   D4's list of what a watch entry holds leaves out the gate-hold key that D6 now stores there.
   Evidence: D6 points to D4 for where the key lives, but D4 lists the entry's fields and the key is not among them.
   The list also does not say whether a run whose `run_watch` is `off`, or has never read, gets a watch entry at all. D2's new sentence says such a reading "neither posts nor becomes the memory", yet D6 still needs somewhere to keep the key for that run.

2. LOW — D4's rationale "the run notices keep their memory the same way", left unchanged beside D4's rewritten storage.
   The rationale no longer matches D4: D4 now puts the watch memory in a map of its own, outside the run's entry.
   Evidence: the run notices keep their memory in `RunState.Notice`, inside `s.Runs[slug]` (`internal/dobbyengine/state.go:86-87`). That is exactly the entry `recordOwed` deletes, which D4's fix avoids. D4 now follows `LandNotices` instead, so "the same way" points at the wrong precedent.

No further tool calls made.
