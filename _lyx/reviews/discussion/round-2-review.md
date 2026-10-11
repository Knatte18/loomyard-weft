---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md D3, D4 and Acceptance criteria
    summary: A required advisor that ends the step early is never itself stopped, though today's decideRound stops a timed-out or wait-errored reviewer and the seat keeps its pane
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D10 Bound
    summary: D10's bound on the chair-notice deferral is vacuous, since the chair starts first under the same Table.Timeout and so reaches its own deadline before any advisor's
---

### [MEDIUM] The failing required advisor is never stopped

**Location:** `decision-record.md` D3 ("the engine stops the chair and every other started advisor and returns"), D4's failure list, and the Acceptance criteria rows for D4.

**Issue:** D3's early end stops "the chair and every other started advisor", which excludes the required advisor whose end triggered it.
Today `decideRound` in `internal/burlerengine/handoff.go` stops any half whose wait errored or whose outcome is `OutcomeTimeout`, because shuttle keeps a timed-out run's strand live and a wait-errored run may still be running.
Under seats it is worse than today: `seatSpec` in `internal/seatengine/prompt.go` sets `KeepPane: !isChair`, so the reviewer's pane also outlives a done end whose gate failed or whose `Accept` errored.
The early-return path D3 adds bypasses `seatengine`'s `join`, the only place that stops "every started advisor, a kept done one included".
If the required advisor is left live, `BurlerProducer`'s retry starts a fresh round next to it.
The new reviewer's `burler-advisor-1` strand name is still held, so `checkName` refuses the start, and meanwhile the live reviewer can still write the round's files.
D4 says the failure rules map "unchanged".
The Acceptance criteria rows ("reviewer died or timed out (chair stopped, …)", "reviewer's wait errored (chair stopped, …)", "audit or strict-parse error (error, chair stopped)") pin only the chair stop, so a plan built from them would not test the reviewer stop either.

**Fix:** In D3, state that the step's early end stops every started seat, the required advisor itself included, and that a stop failure there surfaces as `ErrSeatNotStopped`.
Add "reviewer stopped" to each of the Acceptance criteria rows named above.

### [MEDIUM] D10's deferral bound never fires before the chair's own deadline

**Location:** `decision-record.md` D10, the `Bound:` sentence ("an advisor that stays mid-turn defers it at most until that advisor's own deadline, the table timeout").

**Issue:** Every seat's spec carries the same `table.Timeout` (`seatSpec`, `internal/seatengine/prompt.go`), and `Run` starts the chair before any advisor (`internal/seatengine/engine.go`).
So the chair's deadline always comes before any advisor's, and the stated bound never limits anything: an advisor that hangs mid-turn suppresses a stalled chair's notice for the chair's whole run.
The parent then hears nothing until the chair times out.
The bound is also unsound on its own terms.
Shuttle keeps a timed-out run's strand live, so the session-state fold can keep reading a timed-out advisor as mid-turn after its deadline, unless the predicate also drops advisors whose run has ended.
D10 never says that it does.
Under the rubric's attack-surface rule, a downgraded guard (the chair's stall notice) needs a bound that holds, and this one does not.

**Fix:** Give the deferral a bound that can fire.
One option: the predicate counts only advisors whose run has not ended, and the deferral is capped by its own constant, such as a maximum deferral after which a held chair notifies regardless.
Another: record in D10 and Open risks that a wedged mid-turn advisor hides a chair stall until the chair's timeout, and say why that is acceptable.
Either way, add the case to the hold-deferral Acceptance criterion.

## Verdicts on the focus entries

- **D8 and the shared seat directives.**
  Not a defect.
  The focus wording assumes the directives are included unchanged, but the current D8 has both Burler seat stencils include neither directive and state their own channel rules.
  D8 names the conflicting lines: the advisor directive's wrap-up and its unprompted messages, and the chair directive's turn end for an up line and its read-on-exist.
  So no seat prompt carries both rule sets, and the question of which lines Burler overrides does not come up.
  Leaving the directives out is mechanically sound.
  `Table.Validate` and `composeSeatPrompt` require no include, and `seatValues` fills the reserved markers whether a stencil uses them or not.
  The Constraints entry for PATTERN-parent-directive and PATTERN-edit-directive still binds the stencils to render both directives once.
- **D8 and the kept reviewer after acceptance.**
  Not a defect.
  D8 splits its bounds into Go-enforced ones (the skipped-handoff error, and the review bytes compared after the fix) and stencil-only ones.
  The stencil-only bounds are: no file written after the review, no unprompted message, a reply that only explains an accepted finding, and a fixer that acts on nothing beyond the accepted review.
  Open risks names the residue: the kept reviewer's edit tools, its writes to a target file or the fixer-report, and reply-borne findings landing unreviewed.
  That meets the attack-surface rule: the new channel's reach and its bounds are stated, and the part Go cannot enforce is recorded as a risk.
- **D3 and every way a required advisor can end.**
  Partly a defect.
  The ends D3 lists are: non-done, done with a failed gate, a wait error, an `Accept` error, and a start failure (via D5).
  Each of them ends the step, and the Acceptance criteria carry each one, the failed-gate case included.
  The gap is who is stopped when that happens, which is F1.
- **D4 and the reviewer wait-error rule.**
  Not a defect.
  It matches `decideRound`: a reviewer whose wait errored is not `accepted()`, the fixer is stopped and joined, and the round returns `burler: shuttle run: %w` with no outcome.
  D4's "returned wrapped, as today" and the matching Acceptance criteria row agree with that, apart from the reviewer stop in F1.
