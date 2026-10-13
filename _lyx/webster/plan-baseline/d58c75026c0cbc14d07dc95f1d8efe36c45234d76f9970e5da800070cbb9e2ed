# Card 19 — The orch stencil handles every landing notice and catches a dropped one

**Edit:**
- `contracts/stencils/orch/orch-template-role.md`
- `contracts/stencils#wordingClaims`

**Intent:**
D17: `orch-template-role.md` routes every `landed` notice to `## After landing`, and two next-wake board checks and a deploy-recovery check catch a landing whose notice was dropped, reading only durable state so they hold across a refresh, a compact or a handoff.

The run loop's item 4 says every `landed` notice goes to After landing, in place of "a notice naming newly ready tasks", and every other `[batten notice]` is still handled as under Investigating a stop;
`## Investigating a stop`'s second sentence says a `landed` notice follows After landing, keeping "neither is investigated as a stop".

`## After landing` keeps its account of what batten's Land row and Worktree-Teardown do, its failure-notice sentence, its fabric-commit, drive-report and friction-note sentences, and replaces the ready-notice and deploy sentences with:
- a `[batten notice]` reading `<slug>: landed` means the task landed: count it as no longer running, run `lyx board intake`, start each task the notice names after `newly ready:` with `lyx batten run <slug> --window`, and, while the hub's landing `deploy` key is empty, run `update-plugins.sh` by hand;
- at every wake, any task you count as running whose board entry is `done` or gone from the board, or whose pair is gone, is treated as landed and no longer running, since its notice may have been dropped;
- for such a task, read its `<slug>: landed; newly ready:` line in the hub notice log (`.lyx/notices.log` in the prime) and start each task it names that the board still shows ready and that has not started; any other ready task waits for the operator's approval, as the board policy requires, so the fallback never starts a task the operator has not released;
- only while the landing `deploy` key is empty, at every wake, and only for landings into a `require_pr_to_base` branch: compare `origin/prod`, which `update-plugins.sh` moves to the commit it deploys, with `origin/main`, and run `update-plugins.sh` when `origin/main` holds a commit `origin/prod` lacks.
The section's lead-in says a landing needs nothing more from you than these steps.
The text keeps naming no `lyx fabric pull`, `lyx config reconcile` or `lyx board prune`, which the existing `mustNot` rows pin.

`wordingClaims`' `orch-template-role.md` rows change with it:
"A notice naming newly ready tasks goes to After landing" becomes a `must` of "Every `landed` notice goes to After landing" in `## The run loop`;
"A landing needs nothing from you" becomes a `must` of "a landing needs nothing more from you than these steps" in `## After landing`;
"after any landing" is replaced by the deploy-recovery rows below;
and four `must` rows in `## After landing` pin the acceptance criteria: "reading `<slug>: landed`" (every landing notice is handled there); "whose board entry is `done` or gone from the board, or whose pair is gone, is treated as landed" (the next-wake board check); "start each task it names that the board still shows ready and that has not started" together with "any other ready task waits for the operator's approval" (the fallback starts only named tasks); "only while the landing `deploy` key is empty" together with "`origin/main` holds a commit `origin/prod` lacks" (the deploy-recovery check and its baseline).
Each row's `why` names the decision it pins.
The claims test's top-level test count is unchanged.

**ImpactSummary:** one stencil section, two run-loop sentences and their claim rows in `contracts/stencils`; no Go behavior changes, and the deployed orch reads the new text after the operator's next `update-plugins.sh`.

**Commit:** 19: orch stencil: handle every landing notice and catch a dropped one
