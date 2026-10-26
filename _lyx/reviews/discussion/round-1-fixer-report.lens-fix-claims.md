```yaml
lens: fix-claims
outcome: reported
```

Task: check that the facts, bounds and behaviours the fix's new or rewritten sentences claim hold against the code at HEAD.

Most claims hold. One bound does not.

**MEDIUM — D9 hold bullet, the bound "any other failed write leaves a status on the same side of the hold"**
- **Summary:** One unguarded write also crosses the hold, in the other direction. Suppose the driver crashes after Beam-Await's own write naming Parent-Align but before Done. The re-spawned driver re-enters Beam-Await, and the seam writes `running · Beam-Await` warn-only. If that write fails, the entry keeps reading `running · Parent-Align` while the run waits at Beam-Await. The hold then stands with no bound: members are refused, membership is refused and the run takes a `max_runs` slot.
- **Evidence:** `internal/loomcli/wiring.go` `newCommitStatusSeam` writes the board status only when `status != lastBoardStatus`, and only on a transition call. A failed write only warns ("next transition retries"). While the row waits, nothing calls the seam again, so nothing retries. The new D9 text makes only the Beam-Await-to-Parent-Align write gate Done. Nothing gates the re-entry write.

Claims checked and found to hold:
- D4: runproc has only `End`, `EndSparing` and `EndProcess`, and no role-scoped end (`internal/runproc/end.go:56-70`).
- D7: a failed owner read reads as not owned elsewhere and is logged once per episode, as the dobby's own read does (`internal/dobbyengine/loop.go:236-249`).
- D5: `resumeOwnerClaim` exists and writes an owner (`internal/battencli/owner.go:241`).
- D12: the dobby ticks on every `board.json` write (`dobbyengine/doc.go:58`), and it has a state record (`dobbyengine/doc.go:23,62`).
- D9: the run status has the form `<state> · <producer>` (`boardengine/runlock.go:17`), and `RunStatusProducer` exists to read the producer half (`runlock.go:29`).
- D1: "the anchor move the dobbies it ended" matches `dobbyWorktrees` (`fabriccli/anchor.go:328`). A prime move ends the prime's dobby and every local beam's, and a beam move ends only the beam's own. Today the restart for the prime goes through `EnsureDobbies`, which covers every owned beam (`anchor.go:468-470`), so narrowing it to the ended dobbies is a real change of behaviour.
- D9: the fb5c10c3 rule quoted in D4 matches the commit message.

Tool calls:
1. `boardengine` `RunStatus` and `RunStatusProducer`, the peers verb and the `EnsureDobbies` callers: needed to verify the producer-half claim and the anchor-move claim, which my context did not show.
2. `fabriccli/anchor.go` `startDobby` and adopt's `StartDobbies` doc: needed to verify what the anchor move restarts today.
3. `fabriccli/anchor.go` `dobbyWorktrees`: needed to verify which dobbies the anchor move ends, which D1's new sentence depends on.
