```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each Decision's mechanism end to end on one concrete case.

**List**

- **D1.** Case: `lym config beam --set autonomy=true` in beam B, then nested beam C forked from B. C inherits B's file and Finalize's notice leaves the key out. Purpose met. The inheritance is asserted, and I did not walk the fork itself.
- **D2.** Case: a member run of beam B (autonomous) while B's orch is down. The run reads B's value, `true`, but its notices fall back to an ancestor's orch or the prime's, which apply their own value. Purpose not met (F2).
- **D3.**
  - Case 1: an autonomous raddle run with `raddle_auto_approve` forced on. Proposals file with no approval, which contradicts Scope Out. Purpose broken (F4).
  - Case 2: an autonomous drift judge with `judge_interactive` off and `judge_ask_parent` on, and the parent answers late. The judge's rulings stand unreviewed. Purpose not met (F5).
- **D4.** Case: a member pull request opened at head H1. The parent sends findings, the session fixes and pushes H2, then must re-ask. No verb or seat path exists for the session to send a re-ask, and the seat records no head. Purpose not met (F3).
- **D5.** Case: Code-Review's bounce budget runs out at the same row again and again, and the orch settles each time with `lym loom resume` and no decision entry. The class-5 count stays at 0 forever. Purpose not met (F6).
- **D6.** Case: the last member of beam B lands. Beam-Await wakes, the four conditions hold, it posts one line and writes the marker; a later member reopens and lands, and it posts again. Purpose met, given the new seams Plan adds.
- **D7.**
  - Case 1: a drained beam B that is behind its parent with no conflict. Step 1 fires `merge-parent` on a false premise, and a busy parent repeats the cycle. Purpose broken (F1).
  - Case 2: a drained beam with its pull request already open and a new member. `lym beam describe` reruns `lym beam pr update` itself. Purpose met.
- **D8.** Case: presence slack, autonomy on, a severe-halt relay opening `[operator]`. The hook forwards it and drops the unmarked turns. Purpose met on this case.
- **D9.** Case: `--set autonomy=true --layer default` from the prime. Each live orch whose value changed is mailed and its next pointer carries the line. Purpose met for this writer. The other writers are another lens's to judge. The `lym beam pr` help text is a residual (F7).
- **D10.** Case: the glossary gains two lines. Purpose met.

**Findings**

F1. BLOCKING, design. Location: decision-record.md, D7 step 1.
- **Summary:** D7 step 1 rests on a false premise. `lym beam pr open` does not refuse a beam that only lags its parent; it refuses only a conflict or an open merge task.
- **Effect:** the orch runs `lym beam merge-parent` on every lag. That is a needless merge cycle per drain, and while the parent keeps moving, each merge task's landing drains the beam to a new lag. The orch can then loop on merge-parent and never reach step 4's operator notice.
- **Fix:** run merge-parent only on the conflict refusal that `pr open` names.
- **Evidence:** internal/beamcli/pr.go:240-256 (`judgeOpenable`: "A beam only behind its parent passes, since GitHub merges it"); internal/beamcli/cli.go `prOpenCmd` Long ("a beam only behind its parent still opens").

F2. MEDIUM, design. Location: decision-record.md, D2.
- **Summary:** a run reads "its parent worktree's resolved value, the worktree whose orch its notices reach". But `ParentStrand` routes to the nearest *live* orch at or above the parent worktree, falling back to the prime's orch.
- **Effect:** while beam B's orch is down, a run under autonomous B goes non-interactive and its `pr-continue` refuses without a parent answer. Meanwhile its notices reach an ancestor or prime orch whose own value may be `false`, and that orch treats them as non-autonomous: it waits on the operator, does not answer under D4's rules and does not apply D5. The reverse holds for a non-autonomous beam under an autonomous parent. The stated agreement "the run and that orch agree on one value" breaks.
- **Evidence:** internal/orchcli/notify.go:177-182 (`nearestOrchStrand`: "nearest one whose orch live reports live… falls back to the prime's orch strand"); internal/orchcli/notify.go:100-102.

F3. MEDIUM, design. Location: decision-record.md, D4 second and third bullets.
- **Summary:** D4 has every re-ask "after the session's fixes" go "through the parent-review seat with an ask number, as code-review asks do". Walking the steps breaks in four places:
  - The seat is Go code a segment producer or Bouncer runs per round. Its files come from burler's `ParentReviewFilesOf` and its generation from the round-1 focus file, which PR-Review has none of.
  - The loom `parent-review` group holds only `answer` and `await`, so the PR-Review session agent has no verb to send a re-ask.
  - `pr-continue`'s head check needs the head each ask was sent at, which the seat does not record.
  - Today's PR-Review ask is sent once per record by the row (`askParentReview`), and the row "asks and does not gate".
- **Effect:** the step that hands a re-ask from the session to the seat has no mechanism. Adding one is a new verb or routing edge the Decision neither names nor bounds.
- **Evidence:** internal/shedadapters/parentseat.go:21-62 (seat fields, `roundSeed`); internal/loomcli/parentreview.go:349,385 (only `answer` and `await`); internal/loomcli/parentseatwiring.go:91-106; internal/landingshed/prreview.go:356-387.

F4. MEDIUM, consistency. Location: decision-record.md, D3 bullet `raddle_auto_approve` against Scope Out.
- **Summary:** D3 forces `raddle_auto_approve` true so that "a raddle proposal files with no approval" (the D3 acceptance criterion). Scope Out says "Releasing draft tasks, making tasks or promoting notes: these stay the operator's under autonomy."
- **Effect:** an autonomous raddle run ends having made tasks, as drafts, with no operator approval. The end state contradicts the record's own scope. Either D3's bound must say that drafts are the only thing made and release stays the operator's, with Scope Out reworded, or the key must not be forced.
- **Evidence:** decision-record.md:30 and :76 and :86; internal/loomshed/raddlewrite.go:65-66 (auto mode skips the approval).

F5. MEDIUM, design. Location: decision-record.md, D3 bullet `judge_interactive` reads `false`.
- **Summary:** forcing the judge non-interactive removes the operator's seat that "decides each ruling", and D3 counts on `judge_ask_parent` to stand in.
- **Effect:** under `judge_ask_parent`, a parent answer after `judge_parent_timeout_min` is refused and "the judge's rulings stand as written". An orch busy with other halts therefore lets every non-stopping ruling through unreviewed. The forced keys can now skip review on expiry, and D3 states no bound for it. PATTERN-config-layers counts "a timeout whose expiry lets that gate pass" as hollowing a gate.
- **Evidence:** internal/driftengine/config.go:89-94; the contracts/orchrole/orch-role.md section on judge asks ("a late answer is refused, and the judge's rulings stand as written"); pattern/PATTERN-config-layers.md:20-23.

F6. MEDIUM, design. Location: decision-record.md, D5 class 5 and its Bound.
- **Summary:** class 5 counts only `--by parent` decision entries at a row. Many of the settlements the role prescribes record no decision: an exhausted bounce budget (`lym loom resume`), a converged Bouncer (`circling accept` then resume), a lock release, and a resume after a transient stop.
- **Effect:** walking a run whose Code-Review budget runs out repeatedly, with the orch resuming each time, the count never reaches two and the repeat is never relayed. "Class 5 bounds the orch's own settlements at two per row" is false for those settlements.
- **Evidence:** the contracts/orchrole/orch-role.md halt section ("An exhausted bounce budget… `lym loom resume`…"; "Settle it with `lym loom circling accept`… and `lym loom resume`"); decision-record.md:129-137.

F7. LOW, consistency. Location: decision-record.md, D9 (rules named as exceptions) and D7.
- **Summary:** D7 has the orch run `lym beam pr open` and `lym beam describe` on a drained line. The `lym beam pr` verbs' own help says "The orch runs it when the operator says so".
- **Effect:** D9 names only `orch-role.md` and `role-slack.md` as taking the autonomy exception, so the verb help the orch reads contradicts D7 under autonomy.
- **Evidence:** internal/beamcli/cli.go `prOpenCmd` and `prUpdateCmd` Long text ("The orch runs it when the operator says so").

**Tool calls**

1. `sed` of internal/beamcli/cli.go lines 60-140 and 215-240: the help for `pr open`, `pr update` and `describe`, to check D7 steps 2 and 3. The manifest held only `judgeOpenable`'s comment, not the verbs' help.
