---
verdict: CONTINUE
rationale: "Round 1 carries a BLOCKING design finding (seed board push killed mid-flight lands after the rollback) and two MEDIUM design findings (faulted fetch refspec read as two repositories, prime key naming a missing branch), and the fixes were reviewed by no fresh reviewer."
---

## Decision

The rule is `CONTINUE` while the latest round carries a gating-class finding at MEDIUM or worse, or any BLOCKING finding.
Round 1 carries three:

- F1, class `design`, BLOCKING: `pushCloneSeed` treats every board-push error as "the board did not land", restores the records branch and removes the hub,
  while origin's receive-pack can still apply the board afterwards.
  It is PLAUSIBLE, resting on P9's hook log and a plain-git control, and the final step was not watched on the sandbox.
  It cites `PATTERN-single-repo-hub`, so it stays BLOCKING.
- F2, class `design`, MEDIUM: a records store whose `remote.origin.fetch` is gone reads as two repositories; CONFIRMED in P5 and P12.
- F3, class `design`, MEDIUM: a `lym.primeRecordsBranch` naming a missing branch exposes the prime records branch to the orphan sweep; CONFIRMED by dry run in P4.

F4 (consistency, LOW) and F5 (scope, LOW) are not gating.
No finding is relabelled.

## Convergence conditions

- No BLOCKING or MEDIUM finding: not met.
- Live gate green: the facts report red, with `internal/fabriccli`, `internal/fabricengine` and `internal/gitrepo` each failing at setup.
  The hub orch states the cause is a seeded `hardener.packages` list without a `./` prefix, which made go read the packages as std packages, so no test ran.
  The red report is therefore weighed as a setup fault, not as a code failure, and the verdict does not rest on it.
- Reviewer model differs from the previous round's: not applicable in round 1; round 2's reviewer must not be `opus`, the model round 1's reviewer and fixer used.

The fix commits for all five findings landed after the review.
From round 2 on, a fix converges a round only for a key first raised in the closing round; these fixes are new text no fresh reviewer has seen, so none counts as convergence.

## Focus departures

- Archive-tag question settled from code (P13), not driven: ratified.
  `add`, `push` and `remove` are refused to a task session, and the review says so and labels the result code-only.
  Round 2 carries the question forward as undriven.
- No concurrency scenario driven, so no concurrency claim and no second-hub proof owed: ratified.
- Sabotage proofs only for the namespace parameter, the prime key, the layout read and the seed order: ratified.
  The lease and the existence check before a delete stay unproven.
- Every focus entry needing `clone`, `add`, `push`, `pull`, `sync`, `reconcile`, `checkout`, `remove` or `cleanup --apply` reported as could not verify: ratified as honest.
  It is a method limit, not coverage: seven of the twelve focus entries were not driven on the substrate.

## Coverage

Not driven in round 1: every pair verb, the join, every refusal, two clones of one origin, the remote delete, the warp-origin sweep (it needs a GitHub URL),
and the lease against a moved origin.
The method also failed to show P9's last step, the board landing after the rollback, because the sandbox removed its origin before the hook returned; it rests on the plain-git control.
The orch also states that seven of the twelve focus entries went unverified because the task-session guard refuses the mutating fabric verbs; they count as not hardened.
