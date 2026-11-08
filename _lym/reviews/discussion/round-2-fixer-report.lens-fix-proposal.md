```yaml
lens: fix-proposal
outcome: reported
```

Fix-proposal lens: I challenged each proposal the fixer carried out as proposed (F1–F8) and looked for a concrete case where it breaks.

1. **LOW: D9's added sentence.** The text is "The rule this rewrites is the Motif-first rule that commit c8e43ac4 landed on main, so the run takes main's text of both files before editing them."
   - Problem: this comes from F6's fix, and it ties both files to c8e43ac4. That commit touched only one of them.
   - Evidence: `git show --stat c8e43ac4` lists only `plugins/ly/skills/board-policy/SKILL.md` (7 insertions, 1 deletion). It does not touch `contracts/orchrole/orch-role.md`, so the Motif-first rule in board-policy is the only text the commit landed. Taking main's text of orch-role.md does no harm, but the sentence is false for that file.

2. **LOW: D6 admission, the new `Raddle-File` producer limit (F5).** A failed status write blocks the filing.
   - Case: loom's status hook writes `RunStatus(state, "Raddle-File")` on the transition into Raddle-File. If that write fails, the hook only logs and continues to the engine (`internal/loomcli/wiring.go:320-326`: "board status write failed, next transition retries").
   - The entry then still names the previous producer, such as `Raddle-Bouncer`. The run lock refuses the filing write, because the before-entry's producer is not `Raddle-File`.
   - That refusal is not `ErrRaddleRecheck`, so Raddle-File returns a plain error (`internal/loomshed/raddlefile.go:242-246`) rather than a Stuck message.
   - Result: a transient board-write failure becomes a failed filing. The record does not name this, and the D6 criterion on the producer limit does not cover it.

3. **LOW: the D2 criterion as F1 rewrote it.** It says "a spent-budget round whose open findings include a design-wrong one does not reach Raddle-File from that round: it continues automatically or escalates to the run's parent."
   - Case: at the hard cap or on circling, the round escalates with `escalate_decisions` on. The arbiter or the parent can then settle that escalation as an accept, and Raddle-File runs from that same round.
   - The criterion therefore holds only for the Bouncer's automatic settle, not for the run as a whole. A test written from it could not pass once the escalation is answered with an accept.

F3 (no Subject seam) holds against the code. A recipe row with no `subject` key gets no Subject, which "keeps it on the clear-and-reseed path" (`internal/shedrecipe/entries_bouncer.go:234-245`), so the split perch's re-entry over a settled verdict clears its run directory. I found no breaking case for F2, F4, F7 or F8.

Tool calls:
- `internal/loomcli/wiring.go:960-1000` and a grep for `DocumentSubject`: my context did not show how a Bouncer row gets a Subject seam (F3).
- `internal/loomshed/raddlefile.go:225-265`: my context did not show how Raddle-File routes a `FileRaddle` error (F2, F5).
- `internal/shedrecipe/entries_bouncer.go:225-262` and a grep of the recipes for `subject`: to confirm that an absent `subject` key gives no seam (F3).
- `git show --stat c8e43ac4`: to check which files the commit cited in F6's fix touched.
