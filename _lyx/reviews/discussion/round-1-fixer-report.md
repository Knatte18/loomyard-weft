# Round 1 fixer report — discussion

## Changes

- F1 (BLOCKING), fixed differently in one part: D2's way forward now reads "commit or discard them in `<checkout>` with plain git", with the evidence against `lyx fabric commit` (`ScopedPathspec`, `Commit`, `sandboxedRoleRefusal`) in D2's rationale, and the note that a commit there reaches origin with Finalize's parent push.
  The proposal named "the operator or orch" as who clears the dirt; the record names the operator alone, with the orch relaying the reason and running the goto and resume afterwards.
  Reason: the parent's code checkout is not the orch's own worktree, and CLAUDE.md's Worktrees section says never to edit, commit or push in another worktree; discarding also throws away the operator's work (self-check, fix-proposal lens).
  D4 and D5 reuse D2's rendering function, so they inherit the change.
- F2 (MEDIUM): new D5, a told check that a Go driver runs on the current row before its first step when its first status read finds the run blocked with no open attempt to resume. `loomcli` wires it to D4's Finalize arm alone.
  The evidence cited is `binaryEligible`/`decideDrive`, `run`/`handOver` and `Step` on a blocked run.
  Scope In, Goal, Constraints and the acceptance criteria were updated for it, and support-log Q10 was added.
- F3 (MEDIUM): D4 dropped its second target kind. The evidence (`appendHistory`, `gotoRows`) was added to D4. Acceptance criteria 5 and 8 were narrowed to Finalize. The support log's rejected alternative and Q7 were corrected, and the reversal is recorded under Review rounds.
- F4 (MEDIUM): D2 renders the path list and the discard way forward only when the error lacks `merge already in progress`, and otherwise renders the error's own text. D4's Bound was corrected, and an acceptance criterion was added.
  The self-check gave that branch its own way forward (let the merge in progress conclude, then goto and resume), so it meets the record's way-forward constraint.
- F5 (MEDIUM): D4's Bound now places the refusal in the refusal spec's transient class and says why it halts rather than warns, and "advisory" is gone. D5 carries the same class.
- F6 (LOW): D2's rationale now gives the true reason for the goto: D4 guards it, so it fails fast while the parent is still dirty. The support log's rejected alternative cites `attemptClosed`, `resumeOpenAttempt` and `Step`.
- F7 (LOW): D2 renders each path Go-quoted, joined by a comma and a space, citing `stuckReason`/`oneLine`. Acceptance criterion 1 names the form.
  This replaces the backtick example the proposal gave: Go quoting also escapes a path holding a backtick or a line break.
- F8 (LOW): acceptance criterion 1 now requires every dirty path up to D2's cap.
- F9 (LOW): D1 now cites the `fslink` directory link made in `junction.go`, `wireJunctionSet`'s exclude entry (`/_lyx`), and the guard's tracked-only scope.
- F10 (LOW): the goto-then-resume fact moved into D2 as settled, with `gotoAdmitted`, `Goto` and `resume.go` cited. Its Auto-mode assumptions bullet was removed.
- F11 (NIT): the note now names the `loomCLI.recordedParentBranch` seam and `fabricengine.OpenParent`.
- F12 (NIT): the sentence moved from Scope's Out to the support log's Turn 3 goal-scope bullet.

## Self-check

- fix-consistency, MEDIUM, the both-reasons branch had no way forward, against the Constraints bullet: fixed (merged with the fix-way-forward finding) by giving the branch its own way forward in D2, D4's Bound and the criterion.
- fix-consistency, LOW, the rejected alternative on bare resume was stale after D5: fixed as proposed.
- fix-consistency, LOW, no criterion pinned D5's admits: fixed as proposed in the D5 criterion.
- fix-consistency, LOW, Turn 3's "assumption is restated" was stale: fixed with a parenthetical pointing at round 1's move.
- fix-way-forward, MEDIUM, the both-reasons stuck reason led nowhere: fixed (merged, see above).
- fix-proposal, MEDIUM, F1's proposal had the orch commit or discard in the parent's checkout and left the ungated push unbounded: fixed by naming the operator alone and stating that the commit is pushed with no gate run of its own.
- fix-proposal, LOW, the Goal's "or driver step" overclaimed for an orch goto onto PR-Gate that routes into Finalize in-loop: fixed by narrowing the Goal and adding the route as an Open risk.
- fix-claims: no findings.

## Disputed

None.

## Deferred

None.
