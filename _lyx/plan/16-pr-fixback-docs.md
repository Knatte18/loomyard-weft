# Card 16 — pr-fixback-docs

**Prosa:**
- `docs/overview.md`
- `plugins/ly/skills/ly-drive/SKILL.md`
- `contracts/specs/loom-status-spec.md`

**Intent:**
Bring the operator-facing docs in line with the gate and `reject` (Task completion rule; D2's hand-made-fix statement).
In `docs/overview.md`'s loom paragraph:
add `reject` to the verb list;
the PR-review wait now halts at `PR-Gate`, not Publish;
`approve` applies to a run awaiting or blocked at the gate and removes a pending rejection;
`reject <review-file>` records the findings (removing a pending approval), refuses once the `PR-Review` segment's five rejection rounds are spent, and `lyx loom start` then routes the run through `PR-Rework`, which appends cards to the plan, re-baselines Webster, and re-runs Webster, `Webster-Review`, `Describe`, `Publish` and the gate;
the landing tail is `Webster-Review`, `Describe`, `Publish`, `PR-Gate`, `Finalize`, with Publish returning Done once the PR is open and refreshing its title and body from the description;
the sentence "`Publish` returns Done on a valid approval (matching the branch's current HEAD) before any sync" becomes `PR-Gate` returning Done on a valid approval, since Publish no longer reads it;
and a run halted at the gate re-runs only the gate on `lyx loom start`, so a fix committed by hand outside loom is pushed by the operator before `approve`, or goes through `reject` instead.
In the module tree, the `internal/landingshed/` line names its three general ShedProducers: Publish, PR-Gate and Finalize.
In the batten paragraph, batten resumes an awaiting child once per operator decision, approval or rejection alike, and the parenthetical "it does restart an approved `awaiting` one" names an approved or rejected one.
In `loom-status-spec.md`, the `error` field comment's "awaiting is the planned PR-review hand-off at Publish" names `PR-Gate`.
In `SKILL.md`, the awaiting-baseline sentence says the operator resumes after approving or rejecting, and the awaiting hand-back wording states the same hand-made-fix rule.
Write every changed line with semantic line breaks.
