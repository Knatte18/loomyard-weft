# A PR-Gate reject with a small fix re-runs the whole darn review segment

`pr-gate-reject-small-fix` · Note · enhancement, shed, burler

## Seen (gate-fewer-new-binaries, 2026-10-20)

The run had passed its review round, Merge-In and Publish when the alignment agent rejected at PR-Gate with one finding (F1): the decision record's Scope bullet did not name the loom plan verify beside `darn.yaml`.
The fix was a wording change plus two doc lines (`8a442c14`), yet the reject re-spawned Darn-Write and the run went through Darn-Bouncer and a whole new Darn-Burler-Review round, then verify and Publish again.
On OSL-1046 that costs tens of minutes for a fix a reader checks in seconds.

## Operator view (2026-10-20)

A whole new Burler round after such a small fix is overkill.

## Idea

A rework after a PR-Gate reject goes back to PR-Gate through its own light check: the alignment agent, or a single bouncer pass, confirms that the reject's findings are fixed and that the diff since the reviewed head touches nothing else, and only a rework that grows past that, by size or by touching code the review did not see, gets a full review round.
The same may fit loom's PR-Rework.