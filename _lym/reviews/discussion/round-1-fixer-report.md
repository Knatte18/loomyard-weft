---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed-differently
    reason: "Class 5 counts earlier halts at the row in the run's status history (shedengine.HistoryEntry), which lym writes on every halt whatever verb settles it, instead of a --by parent decision entry per settlement. Such entries would land in the decision record that drift and darn read as decisions (internal/driftengine/judgesnapshot.go CheckAddedEntriesSubset), and each would mail the run a decision notice."
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed
  - id: F15
    disposition: fixed
  - id: F16
    disposition: fixed
  - id: F17
    disposition: fixed
  - id: F18
    disposition: fixed-differently
    reason: "F13's fix drops D9's change mail: the orch reads its autonomy at call time, from the halt notice and lym config beam --resolved. A bullet on which orchs a --layer default write mails would test a mechanism the record no longer has. It is replaced by a bullet saying a change made outside --set reaches the next halt notice."
  - id: F19
    disposition: fixed
  - id: F20
    disposition: fixed
  - id: F21
    disposition: fixed
---

## Changes

Every finding's premise held against the code cited.
Changes are to `_lym/discussion/decision-record.md` unless named otherwise.

- F1: D7 step 1 runs `lym beam merge-parent` only on `lym beam pr open`'s conflict refusal (`judgeOpenable`).
  A beam only behind its parent goes to step 2, and step 3's conflict refusal sends the orch back to step 1.
  The Review D7 bullet matches.
- F2: every halt notice carries `autonomy: <true|false>`, the run's value.
  The orch handles a halt under that value, not its own.
  A new D2 Bound states the down-orch case, the agreement claim in the Rationale is replaced, and a D2 bullet and a Review D2 bullet are added.
- F3: D4 names `lym loom parent-review ask <slug>` (new).
  Ask 1 comes from the row, and each re-ask from the session agent.
  The pull request record keeps each ask's number, the head it was sent at, and its answer.
  The verb's bound is that it is refused outside a PR-Review session under autonomy and while an ask stands unanswered at the current head.
  The Constraints shellguard line names it, and a D4 bullet is added.
- F4: under autonomy the `[pr-review request]` line names `lym loom parent-review answer <slug> --ask <n>`.
  D9's exceptions now cover `orch-role.md`'s "Reviewing a pull request". A D4 bullet and a Review D4 bullet are added.
- F5: under autonomy, `pr-continue` with a mailed head standing and no satisfied answer clears the mailed head and leaves the review open.
  On a closed pull request it checks no answer. A D4 bullet is added.
- F6: PR-Review records the autonomy at session start in the pull request record, and that value governs the review to its end. A D4 bullet is added.
- F7: a D3 Bound says that under autonomy an expired judge ask stops the run at Drift-Verify instead of letting the rulings stand. A D3 bullet is added.
- F8: D3 forces the run's setting lookup too.
  The restore sentence names Raddle-Write's mode, which is fixed at the first call, as the exception. D3 bullets are added.
- F9: Scope Out now reads "Releasing draft tasks, filing tasks of the operator's own and promoting notes" and names D3's drafts and D7's merge task.
  D7's Bound names merge-parent as the one work-starting step the line licenses, and D9 names `orch-role.md`'s go-ahead rule.
- F10 (fixed differently; see the frontmatter): class 5 is "a halt at a row of a run whose status history already holds two earlier halts at that row".
  It is counted from `shedengine.HistoryEntry`. The Bound and the Review D5 bullet match.
  I did not make the proposed `--by parent` entry per settlement.
- F11: class 6 covers a halt the orch finds no settling action for.
  The Bound says every halt has a route, and the Review D5 bullet says six classes.
- F12: D6 keeps the drain marker in a file of its own under `.lym/beam/`, named in `landrequest.go`, which Beam-Await alone writes and removes.
  The Bound names the extra line a takeover or new pawl run may post, and the D6 bullet adds the dobby-down notice cycle.
- F13: D9 drops the change mail.
  The orch acts on the value read at call time: from the halt notice, or from `lym config beam --resolved` for its own D7 steps.
  D8's marker no longer depends on any reading of autonomy (see F14), so the hook and the orch cannot disagree.
  A D9 bullet covers a change made outside `--set`, and the Rationale lists the writers.
- F14: D8 has the orch post each severe-halt relay and merge-ready notice to `<shortname>:SLACK` directly, with one relay per turn opening its own reply.
  While the presence is slack, the orch marks every operator reply whatever its autonomy.
  A Review D8 bullet is added.
- F15: D2's Rationale drops the claim that the orch hears of a broken `beam.yaml`.
- F16: D9 names the pointers that carry the line: start, resume, reload, and the pending resume after a handoff. The D9 bullet matches.
- F17: D9's exceptions now cover the Long help of `lym beam pr` and `lym beam describe` in `internal/beamcli/cli.go`, and the Review D9 bullet matches.
- F18 (fixed differently; see the frontmatter).
- F19: D5 files the bug with `lym selfreport create`, never as a task, and the shellguard constraint names `selfreport`.
- F20: D1 pins the inheritance to the weft branch's fork point.
  The pin is `fabricengine.ReadConfigChanges` and its row "a file present at the fork point carries its base bytes".
- F21: Auto-mode assumptions reads None.
- `support-log.md`: Round 1 is logged.
  The rejected alternatives gain the change mail and the orch's own value, and Q5 and Q11 are updated.

## Self-check

All four forks returned a report.
Further changes from the self-check, beyond the F10 change above:

- fix-consistency, MEDIUM, D5 class 2 contradicts Scope Out: fixed. Class 2 reads "a new task the orch would have to file, since it files none beyond D7's merge task".
- fix-consistency, MEDIUM, D8's marker rule belongs in `role-slack.md`: fixed. D9 puts it in `role-slack.md`, and `role-autonomy.md` holds the direct posts.
- fix-consistency, LOW, D2's "never captured" contradicts D4: fixed. D2 names PR-Review's value, recorded at session start, as the one exception.
- fix-consistency, LOW, turn 3's lines in the support log are stale: fixed. Round 1 names them as superseded.
- fix-claims, MEDIUM, `--print` is not the resolved value (also fix-way-forward 2): fixed. The text now reads `lym config beam --resolved`.
- fix-claims, MEDIUM, resume does not re-ask the judge's parent (also fix-way-forward 3): fixed. D3 says the resume clears the timed-out ask and sends a new one over the same rulings, marks this as new, and adds a D3 bullet.
- fix-proposal, MEDIUM, `--by parent` entries pollute the decision record: fixed differently.
  Class 5 counts from the status history (F10 above).
- fix-proposal, MEDIUM, a toggle cannot reach a running review: fixed. A goto that re-enters PR-Review records the value anew. A D4 bullet is added, and the support log's trade-off is updated.
- fix-proposal, LOW, class 2 is stale: merged with the fix-consistency finding above; fixed.
- fix-proposal, LOW, a relay reaches Slack twice: fixed. D8's Bound states the duplicate as the price of a relay never lost.
- fix-way-forward, MEDIUM, `pr-continue` says ask again while an ask is waiting: fixed. With an ask unanswered at the current head, the report and the refusal name waiting for the answer. A D4 bullet is added.
- fix-way-forward, MEDIUM, `gateRetryHint` names the operator: fixed. Under autonomy D4 drops the hint's operator condition. A D4 bullet is added.
- fix-way-forward, MEDIUM, `--print`: merged with fix-claims; fixed.
- fix-way-forward, MEDIUM, the judge re-ask: merged with fix-claims; fixed.

## Disputed

None.

## Deferred

None.

## Rulings

None.
