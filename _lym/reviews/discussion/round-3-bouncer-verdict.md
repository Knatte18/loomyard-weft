---
verdict: CIRCLING
rationale: "F19 (BLOCKING design) carries F17 on key landing-commit-no-durable-source, open in rounds 1 and 2 and still open: D5 can still name another run's commit, or none; F20 and F21 are new BLOCKING design findings"
---

The latest round has three BLOCKING `design` findings, so the loop has not converged.

The verdict is `CIRCLING` rather than `CONTINUE` because F19 maps to `landing-commit-no-durable-source`,
which the facts list as open in rounds 1 and 2, and the review marks F17 still open.
The fix did make progress: the review says the parent's head is no longer read on these paths.
The replacement search still fails on the same point, according to the review:
Finalize's own merge-in moves the task tip it searches over, and a squash landing breaks its premise.

The other keys that were open after round 2 are resolved (prior F16 and F18).

- F20 is a new key: D5 trusts a landing record after the parent was reset past the merge it names.
- F21 is a new key: D1's tracked `.gitignore` on the records side violates `PATTERN-fabric-boundary`.
  It stays BLOCKING and gating.
  The review notes that it sits on a hunk the last fix did not change, which the reviewer's incremental rule would bar.
  The PATTERN entry wins over that rule.

No finding is relabelled.
The review declares no focus departures.
