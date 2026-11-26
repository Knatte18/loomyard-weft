---
round: 2
focus:
  - "In _lym/plan/03-run-watch-engine.md, trace every path on which a done run still owes a landed line (watch level not yet read, state record that reads but does not write) against clearDoneRuns in internal/dobbyengine/donerun.go and the scheduling block of tick in internal/dobbyengine/loop.go: can the run be cleared before the line is judged and posted, and does a named test reach each hold the card states?"
  - "In _lym/plan/03-run-watch-engine.md, follow the watch entry's copy of the land_notice reading through a level that first reads after Land and through a tick that leaves the run unlisted, against rememberRuns and unreadLandNoticeRuns in internal/dobbyengine/landnotice.go: for each sequence, does the orch get exactly the landed line or landing notice the decision record asks for?"
  - "In _lym/plan/03-run-watch-engine.md, check what StatusSeen and ChildSeen hold after a failed ReadChild and whether a later tick with unchanged status modification times reads the child again and posts its stop."
  - "In _lym/plan/06-gate-hold-engine.md, compare the processes the gate-hold line names with D6 and its acceptance criterion in _lym/discussion/decision-record.md: is any narrowing of that list licensed by the record?"
---

These entries cover the memory and clear behavior of the run watch, and the content of the gate-hold line.
