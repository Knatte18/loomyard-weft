```yaml
lens: fix-proposal
outcome: reported
```

fix-proposal lens: I challenged each reviewer proposal the fixer carried out and found two concrete cases where it goes wrong.

1. **MEDIUM.** The F2 change, D2's new bullet ("An escalation raised again is answered as any other: `lym loom circling continue` runs the next round over the new proposal") and the new D2 criterion ("the perch raises that escalation again, and `lym loom circling continue` then runs the next round over the new proposal"), break when the escalated round is at or past the hard cap.
   - At the hard cap the re-raised escalation offers only `lym loom circling accept`. A `continue` there is retired and the round escalates again with cause hard-cap.
   - So a goto from a perch halted at the hard cap has one answer, and that answer files the new proposal with no round having reviewed it. The bullet presents `continue` as always available, and the criterion fails for a hard-cap escalation.
   - Evidence: `internal/shedadapters/bouncer.go:1603-1604` ("the round is at or past the perch's hard cap, so it can only be accepted") and `bouncer.go:1257-1261` (a continue at the hard cap goes through `retireContinue`, then `escalate(round, EscalationHardCap, ...)`).

2. **LOW.** The F4 change, D6's new halt ("When the entry is already of kind `project` and neither test holds, it files nothing and halts for the parent ... naming the entry's `order` as a filing the run did not make"), mislabels one reachable case.
   - The case: the run files its family, which converts the entry, and the store write fails. A `lym loom goto --to Raddle-Write` then re-splits; nothing in D7 or `CheckGotoTarget` forbids it while the run stands at Raddle-File.
   - The new proposal's hash then differs from the staged hash and from the hash in the store. Neither `StoreDone` nor the stage-hash plus `AlreadyFiled` test holds, so Raddle-File halts and calls the run's own filing one "the run did not make".
   - The way forward still works, but the stated cause is false. The halt should name the filing as one not made from the current proposal.
   - Evidence: `internal/loomshed/raddlefile.go:137-149` (both tests key on the current proposal's `hash`) and `internal/shedengine/goto.go:315-361` (`gotoAdmitted` admits rows before the current one).

The other proposals hold up. For F1 (the goto test), F3 (the soft-cap wording) and F6 (the retry rule), I could not build a case where the change is wrong. For F5, the lock reads the parts in the sequence they appear in `after`. That holds only if the board's filing write appends the parts in filed order. I found no sort in the write path, so it stands.

Tool calls:
- A grep of `FileRaddle` and the sort calls in `internal/boardengine`, to check whether the board's filing write keeps the part order. My context lacked the board write path.
