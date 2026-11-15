```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked every pair of decisions that act on the same state.

**Pairs checked**
1. D1 act rule (a landing past its parent-side merge gets `act`, whoever owns it) × D4 hand-over: **broken** (F1)
2. D1 × D8/D7, from the adopter's side (the adopter owns the landing run, which nothing holds): **broken** (F2)
3. D2's synchronous transition push × D3 pause intents and D7 `hold` entries, which feed the network gate: **broken** (F3)
4. D3 batten-record removal at teardown × D1 `landed` and `queued` × D11 teardown of landed runs: **broken** (F4)
5. D11's teardown condition reading the act rule × D1's `wait` for terminal states: **broken** (F5)
6. D10 restore rule × the D4/D7 window in which the old owner still mirrors: **broken** (F6)
7. D9 ends at hand-over × D10's final mirror at orch stop: covered by the D10 bound for the post-takeover turns only; the pre-takeover gap is outside this lens and left to the handler and walkthrough
8. The `PATTERN-refusal-way-forward` constraint × the refusals D3 and D5 add: **broken** (F7)
9. D2 × D3 offline verbs: held (the bound says so)
10. D3 offline resume × D1 owed-push `wait`: held
11. D3 offline intent × D4 dropping it at hand-over: held (the bound and the notice cover it)
12. D3 × D6 step 5 (the owner writes pause entries from intents): held
13. D3 `lym tender resume` × D7 `hold` entries: held
14. D1 `lym batten cancel` setting a hold aside × D7 "cancel goes on under a hold": held
15. D7 retiring the owner pause × D9 ends: held
16. D1 rule order (`hand_over` before a hold's `wait`) × D7: held
17. D7 parent-lock exemption × D1 starting no producer: held
18. D6 steps 3 and 4 × D4 hand-over and D8 step 2 reset: held
19. D8's re-read of its own hold × `--release` × a second adopter: held
20. D8 step 8 `adopt-fix` × D7 fix-task `act` × D2 synchronous claim: held
21. D5 registry × D8 step 1 one-commit write × D7 hold machine names: held (the refusal's way forward is F7)
22. D3 cancel-with-keep × D1 `cancelled` × the need to remove before re-releasing: held (today's cancel already demands `--remove` before a re-run)
23. D11 upgrade migration × D1 `queued`: held
24. D6 step 4 × D11 orphan detection: held (a board entry keeps the pair out of the orphans)
25. D10 session record on a records branch × D4 rejection: held (the same session id on both machines)

**Findings**

**F1. BLOCKING, design. D4 hand-over × D1/D6/D9 landing carve-out.**
- Problem: D8 step 3 writes the takeover entry on every moved run, a landing past its parent-side merge included. On the old owner, that Finalize's next records push is rejected and D4 hands the run over: it ends the processes, pushes nothing and archives.
- So D4 kills the very landing that D1 (`act` when another machine owns it), D6 step 4, D9 and the PATTERN-owner-machine constraint say finishes there. D4 has no carve-out.
- The D6 criterion "keeps its processes and pair through the catch-up" holds only until the next push.
- Evidence: decision-record.md:147-152, :52-53, :193, :300, :378-379, :430.

**F2. BLOCKING, design. D1 act rule × D8 retiring the adopt pause.**
- Problem: after a takeover, a run past its parent-side merge reads `act` on both machines. The old owner gets `act` because the landing carve-out ignores the owner. The adopter gets `act` because it owns the run, and D7 holds no landing.
- D8 lets moved runs go on with no hand lift, so both machines drive the same Finalize at once. The parent lock is machine-local and cannot keep them apart.
- Today the adopt pause kept the adopter from driving it.
- Evidence: decision-record.md:52-56, :212, :269; state map 3.2 parent lock "valid only while the owner process lives"; PATTERN-dobby-runs "Adopt pause".

**F3. MEDIUM, design. D2 synchronous push × D3 pause intent and D7 `hold` entries.**
- Problem: D2 retires fabric's pause deferral for a transition's push and claims each transition is on origin before the run acts. Yet D3 and D7 still enforce pauses and holds through the pause record's network gate, and every git network call of a run's process passes that gate (`pauserecord.Network` in gitexec, `Admit` under the remote-refs lock).
- A pause or hold that lands between a transition's commit and its push holds the push. The state then stays local for the whole pause or hold, which is exactly the moment the operator stops and moves to another machine.
- D2 is silent on whether a transition's push is exempt from the gate.
- Evidence: decision-record.md:74-76, :89, :108, :213-216; internal/gitexec/gitexec.go:254; internal/gitrepo/remoterefslock.go:34; internal/pauserecord/doc.go "Network is the process gate...".

**F4. MEDIUM, consistency. D3 "the batten record goes with the pair's teardown, D11's" × D1 states × D11.**
- Problem: D11 tears down landed pairs. If the batten record goes with that teardown, a landed run afterwards has no batten record and a board status of `done`, so it matches none of D1's states. `landed` needs the record, and `queued` needs an empty status.
- If the sentence is meant for cancels only, D1 never says what a landed run reads after teardown either.
- Evidence: decision-record.md:35, :43, :105, :355.

**F5. LOW, consistency. D11 teardown condition × D1 act rule.**
- Problem: D11's teardown "conditions read the FSM state and the act rule". D1 answers `wait` for every terminal state, so a condition on the act rule never fires for `landed` or `cancelled`.
- The record does not say which answer gates the teardown.
- Evidence: decision-record.md:55, :356.

**F6. LOW, design. D10 restore rule × the post-takeover window.**
- Problem: the old owner's late mirror can land after the adopter has restored and resumed; D10's bound admits this.
- If the adopter's orch restarts before its own next mirror replaces that one, the mirror names another machine. D10 then restores the old owner's mirror over the adopter's own newer local transcript, and the resumed session loses the adopter's post-adopt turns. They are kept only beside the transcript under another name.
- Evidence: decision-record.md:329-331, :336-337.

**F7. LOW, consistency. `PATTERN-refusal-way-forward` constraint × D3 and D5.**
- Problem: the constraint covers only the refusals the act rule, the holds and D9 add. D3's refusal of pause or cancel of a run another machine owns names only the machine. D5's refusal of a clashing registration, which fails D8 step 1's owner write, names only the machine too.
- Neither refusal names a way forward, and the constraint does not cover them.
- Evidence: decision-record.md:112, :169, :389-390.

**Tool calls**
1. grep for `pauserecord.Network` and `Admit` in gitrepo, gitexec and fabricengine: my context showed the gate's doc but not whether git pushes pass through it, which F3 needs.
