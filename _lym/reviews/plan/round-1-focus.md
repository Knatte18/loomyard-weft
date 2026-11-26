---
round: 1
focus:
  - "Card 3, child re-read gate: what StatusSeen and ChildSeen hold after a failed ReadChild or a found=false answer, and whether a child change can then be missed until a status file is written again (D1). Also whether the tests as listed would catch it."
  - "Card 3, the watch memory against runfsm and battencli: whether StatusModified and ChildStatusModified move on every change of the inputs of the reading, and whether HistoryLen plus LastRepeats gives a distinct episode for each second stop the Added after Discussion entry covers, including one where the stopped layer switches between batten and child."
  - "Card 3, land_notice copy and landed skip: trace internal/dobbyengine/landnotice.go, countLandings and donerun.go to settle whether the copy exists when D3 needs it, including a run whose level was never read (the skip of the whole run in step 1 against step 2's copy), and whether the done-run clear can drop a run before its landed line is owed, such as with an unread level, a standing anchor move or a watch step that ended on a state error."
  - "Cards 3 and 6, shared RunWatchState entries: an entry created by the gate check with no Last, entry upkeep and pruning when the watch seams are nil or the run is unlisted, and whether GateHolds keys survive every path D4 and D6 require (a failed TaskWorktree, a run left out of one listing, a done run in listedRuns)."
  - "Card 6, holder matching and process naming: how Holder.Worktree as written by internal/gatecli acquireSlot compares with the path pairLocation resolves (symlinks, anchor subdirectory), and whether Tree and ProcessSample in internal/proc give what the line needs for marked processes outside the tree. Check too that dobbyengine may import internal/proc."
  - "Card 5, marker: whether runTest and acquireSlot in internal/gatecli run in the same process for every tier run, detached and nested runs included, so the recorded marker is the entry the run's processes carry (D6)."
  - "Cards 1, 4, 6 and 7, tests: for each planned new test or row, whether an existing test already covers the behavior, and whether the claim that no test calls CheckLocalValues leaves it covered through another surface such as internal/configreg."
  - "Cards 1 to 7, fidelity: every Decision, Constraint and acceptance criterion of the decision record has a carrying card and a reaching test, notably D5's autonomy stamp and untagged post, D6's one Warn per failure episode and hub keys, and D7 across doc.go, template.yaml and orch-role.md. Check whether internal/beamcli/doc.go, listed under Uses by cards 4 and 7, describes the dobby wiring or notices in a way the cards falsify."
  - "PATTERN fit: check the cards against PATTERN.md and its background for config-layers, config-key-names, refusal-way-forward, agent-addressing, gate-slot, sole-parsers, tolerant-records and test-speed, including whether the new probe-tick work and /proc reads respect daemon-wakeups."
  - "Cards 3, 6 and 7, bounds and granularity: whether each new post path and guard states what it can let through and what bounds it, and whether card 3 and card 6 are each one independently reviewable unit with the Create and Edit targets matching their Intent (member glyphs, Extends on doc.go)."
---

Seed focus for round 1.
The lens-review files in the plan directory are not in the Card Index and are outside the subject.
