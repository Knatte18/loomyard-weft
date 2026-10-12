# Card 26 — orch-stencil-pr-gate

**Edit:**
- `contracts/stencils#wordingClaims`

**Prosa:**
- `contracts/stencils/orch/orch-template-role.md`
- `docs/overview.md`

**Intent:**
Bring the orch's role stencil in line with the merge review and the parent lock (D13).
The `## PR-Gate` section tells the reviewer fork to judge the run against its board goal and the operator's recorded decisions, to leave code review to the Webster and Merge reviews, and to read the carried-over `Merge-Review` open risks in the decision record as part of the run.
Checking the run against other runs in flight stays with note `alignment-agent`, so the stencil does not ask for it.
The stale lines change:
- the run-loop exception and the escalation bullet name the first verify failure at `Merge-In` (the plan verify) or `Publish` (`publish_verify`), which batten retries itself, and the goto stays `Webster-Burler-Review`;
- the Finalize line says Finalize lands the task branch under the parent lock, and that a parent that moved since Merge-In sends the run back to Merge-In by itself;
- a new escalation bullet names `lyx batten lock list` to see who holds a parent lock and `lyx batten lock release <branch>` to free one whose owner is alive but wedged, and states that a release never stops a run.
`wordingClaims` follows the changed lines: the retry claims name Merge-In beside Publish, the `goto --to Plan-Bouncer` ban stays, and a claim pins that the PR-Gate fork leaves code review to the Webster and Merge reviews.
`docs/overview.md` describes loom's landing tail as `Describe → Merge-In → Merge-Review → Publish → PR-Gate → Finalize` and darn's as `Describe → Merge-In → Publish → PR-Gate → Finalize`.

**ImpactSummary:** the orch stencil's PR-Gate and escalation text and its pinned claims; no code path changes.

**Commit:** 26: orch: PR-Gate fork judges goal and decisions; stale landing lines updated
