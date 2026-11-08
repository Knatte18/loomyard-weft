```yaml
lens: fix-way-forward
outcome: reported
```

Task: check that every way forward the fix introduced can be carried out from the state it names and makes progress.

**[MEDIUM] D6 has no way forward when the producer gate refuses Raddle-File's own filing**

The changed text in D6 reads: "The run lock admits the change only when the entry before the write holds a run status whose producer is `Raddle-File`". The Bound reads: "only while the entry's run is at Raddle-File".

Loom's status hook writes `RunStatus(state, producer)` on each transition. When that write fails, it only logs and leaves the retry to the next transition: "board status write failed, next transition retries" (`internal/loomcli/wiring.go`, the `SetBoardStatus` branch).

If the write at the move into Raddle-File fails, the entry still shows the earlier perch row's producer when Raddle-File makes its filing write. The new producer gate then refuses Raddle-File's own filing as a run-lock refusal. D6 names no way forward for this case. The refusal takes Raddle-File's `on_stuck` back to Raddle-Write. That forces a full re-split and a new perch cycle, and it counts toward Raddle-File's third-failure halt.

D10 also treats a run-lock refusal during the filing as a halt that names the refusing entry. Here the refusing entry would be the arras entry itself, which reads as a wrong way forward.

The fix needs to name the case. One option is for Raddle-File to write its own run status before the filing write. Another is to state that this refusal retries Raddle-File rather than returning to Raddle-Write.

**[LOW] Another writer's filing is not repaired by a Raddle-File "retry"**

The changed text in D6's Bound reads: "another writer that makes this exact write while the run is at Raddle-File also passes and ends the run's hold … Raddle-File's retry then finds the entry filed."

When another writer files first, Raddle-File's next call is its first attempt, not a retry. D6 states the repair (write the mark, clear the run status, write the store) only for "a Raddle-File retry that finds the converted entry as filed". So the text never says that a first attempt finding the entry already converted takes the same path. It also never says whether record files and the raddle store, which the other writer did not write, get written. As written, the sentence points at a step D6 does not define for this state.

The other new ways forward check out:
- **D2 re-arm on a goto:** the Bouncer clears on re-entry over a settled verdict when it has no Subject seam (`settledEntry` returns `entryClear`).
- **D4 soft-cap escalation:** goes to the run's parent through the existing `escalate` path.
- **D6 status clear on retry:** changes only the status, which `scopeChanged` ignores, so the run lock passes it and D8 can then release the project.
- **D9 "take main's text":** c8e43ac4 is on main, so the run can bring it in.

Further tool calls: none.
