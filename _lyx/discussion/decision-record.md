# Decision record — loom-tail-review

## Goal

Rebuild the run tail after the code is written, for loom and darn, as the board entry `loom-tail-review` designs it:
loom runs Webster, then Merge-In with a fixer that also aligns, then the Code-Perch, Describe, a PR-Review stub behind a flag and one locked Finalize block;
darn runs its writer, Darn-Review, Describe, its own PR-Review stub and the same Finalize.
Each recipe fixes its own merge and test failures inside these rows, the parent lock covers only Finalize, and the PR rows, the Merge-Review perch and every route that sends a verify failure to another row go.

## Scope

In:

- `contracts/recipes/loom-recipe.yaml` and `contracts/recipes/darn-recipe.yaml`, their header comments, `internal/loomrecipe`'s routing, budgets and darn rewrite, and `internal/loomshed`'s row names.
- `internal/landingshed`: Merge-In without the lock, the merge fixer session, the PR-Review stub, Finalize's locked block and the removal of Publish, PR-Align, PR-Gate and the pull-request code.
- The Webster-Review perch renamed and moved as the Code-Perch; the Merge-Review perch removed.
- The verify-failure routing removed from every reader (D11).
- The PR and approval surface removed (D13).
- Config: the `loom.yaml`, `darn.yaml`, `landing.yaml` and `gate.yaml` templates, the hub's `profiles.yaml` keys and board `run_settings` keys through one migration (D14).
- Readers updated in the same commit (D13).

Out:

- The parent-review alignment point before the Discussion perch: it stays, with `skip_alignment` and `alignment_timeout_min`.
- The conflict-resolution session (`internal/mergeresolve`) itself: unchanged, only where it runs changes (D9).
- Webster's own gate and its fixer (board entry `verify-fix-escalation`): board item 1 holds already, since the hub's `final` gate lists `[0, 1, 2]` and Webster ends Stuck on a gate it cannot clear.
- The human PR review the stub grows into, and the row entry gates `gate-runs-once` builds.
- The hub-wide orch rules on Bolt (`_board/_lyx/orch/rules.md`) that name PR-Gate, Publish or batten's verify retry: the orch rewrites them through `lyx orch rules` as a step of landing this task, beside the board's pause before Merge-In.
- Every other board entry that names the removed rows, keys or the old darn Merge-In, among them `discussion-alignment-into-lenses`, `speed-loom-profile`, `gate-runs-once`, `regression-baseline-verify`, `loom-beam-darns`, `orch-any-worktree`, `umbrella-daemon`, `fixers-on-sonnet`, `review-setup-audit` and `claude-md-lyx-rules-out`: the orch amends them after landing.

## Decisions

### D1. Loom's tail

Decision: the loom recipe after Webster is `Webster` → `Merge-In` → `Code-Bouncer` ring → `Describe` → `PR-Review` → `Finalize` → `Friction-Reflect`.
Finalize's `on_reroute` names `Merge-In`.
Merge-In and Finalize keep no `on_stuck`: a Stuck they cannot fix halts the run.
The rows Merge-Bouncer, Merge-Burler-Review, Merge-Burler-Fix, Publish, PR-Align, PR-Gate and PR-Rework go, with their engines and their `internal/loomshed` names.

Rationale: the board's order; Describe already precedes the lock, and an in-flight run on a removed row is restarted, not migrated, as the board's landing note arranges.

### D2. Merge-In takes no parent lock and moves no parent

Decision: Merge-In keeps its status commit, its run-scope origin reconcile and its clean-tree checks, takes no parent lock, and writes nothing in the parent pair.
With an upstream and no push skip, it fetches and merges the parent's upstream tip, never the local tip, so a landing another run made locally but has not pushed is never merged in.
With the push skipped or no upstream, it reads the local tip while holding the parent lock only for that read, and merges that tip after releasing the lock.
A missing upstream ref, a fetch failure and a mid-merge task pair stay Stuck with their way forward, as today.
Fast-forwarding the local parent branch moves into Finalize, under the lock (D10).
The conflict session resolves conflicts as today; Merge-In then runs the `landing` gate through `verifytree.Verify` and the merge fixer (D4), and writes the merge record with the tip it merged.

Rationale: the board removes the lock from Merge-In.
The fabric merge-in writes only the task pair's code side, since the weft is never a merge participant (`internal/fabricengine/doc.go`, "The merge surface"), so with the fast-forward gone Merge-In writes nothing parent-side.
The upstream tip is the parent as landed and pushed; the local tip can hold another run's squash whose push later fails and is reset (`internal/landingshed/doc.go`'s repair note).

### D3. The merge base every reader takes

Decision: the local parent branch can now lag the tip a task merged, so every reader of "the task's diff" or "since the merge base" takes the merge base of HEAD with the merge record's merged tip, falling back to the local parent branch only when no merge record exists.
The readers are the Code-Perch's review range and subject seam, Describe's diff, the headroom and comment lints, and D10's overlap test.

Rationale: a merge base against the lagging local branch counts the parent commits Merge-In merged as the task's own.

### D4. The merge fixer: one Opus session, shared by Merge-In and Finalize

Decision: `internal/landingshed` gains the merge fixer, a cold session in its own stencil, read through `internal/stencilstore`, with the parent and edit directives.
It runs at `landing.yaml`'s new `fixer` model-spec, default `opus[high]`, bounded by `fixer_timeout_min`, default 60.
It is told the decision record, the merge record, the plan directory where one exists, and, after a failed gate, the landing failure record (D11) with the gate's log.
It fixes and commits on the task branch, and never pushes.
It ends by writing a report whose verdict is `fixed` or `escalate`; a new spec under `contracts/specs/` pins the report's format and one parser in `landingshed` reads it.
After each turn the row re-runs the `landing` gate; a failed gate re-prompts the same session with the new failure until the row's attempt cap is spent.
The session has two modes, told by its row: `merge-in`, which aligns and fixes, and `finalize`, which fixes failing tests only.

Merge-In spawns it when the `landing` gate fails, or when the parent brought in commits since the merge record's aligned tip, whatever the gate says.
An absent aligned tip, as on a run's first Merge-In, reads as the task's merge base before this merge.
In `merge-in` mode the fixer reads what the parent brought in since the aligned tip against the decision record, fixes what follows from the record, fixes the failing tests, and escalates a conflict between the parent's design and a decision of the record instead of choosing one.
A `fixed` verdict over a passing gate advances the aligned tip to the merged tip, and the row is Done; a merge that brought no parent commit and a passing gate spawn nothing and leave the row Done.
Only Merge-In writes the aligned tip; the merge record's reviewed-tip field becomes it.
An `escalate` verdict halts the run, its reason naming the report and the conflict, with the way forward `lyx loom decision add` in the task worktree to settle it, then `lyx loom resume`; the aligned tip stays, so the resumed Merge-In runs the fixer again over the amended record.
Merge-In's fixer gets `landing.yaml`'s new `merge_fix_attempts`, default 2, gate failures in one row entry; a spent cap halts the run naming the failure record's log, with the way forward to fix on the task branch, then `lyx loom resume`.
Both halts carry the landing-stop reason prefix that D11's driver row hands back.

Rationale: the board asks for one Opus agent, no twill, that reads what the merge brought in against the decision record.
One session re-prompted per failure keeps its context, as the row gates of Describe and the writers do.
The aligned tip, not the last merge, decides what the fixer reads, so a merge Finalize made before routing back (D10) is aligned at the next Merge-In.
Two attempts with an Opus-high session replace three Sonnet rounds, in line with the operator's view in `verify-fix-escalation`.
`landing.yaml` holds the keys because darn and loom share Finalize, and landing config is hub-wide.

### D5. The Code-Perch replaces the Webster-Review perch, after Merge-In

Decision: the Webster-Review perch is renamed and moved: segment `Code-Review`, rows `Code-Bouncer`, `Code-Burler-Review` and `Code-Burler-Fix`, `role_prefix` and `run_subdir` `code`, carry-over `Code-Review`.
Its subject is the task's committed diff at the merge base D3 names.
Its answer key is the decision record and the plan; its rubric, renamed `loom-rubric-code-review`, reviews the code in general and against both, and its review range loses the PR-Gate rework heads and the failure-record merge commit.
The rubric's `publish_failure` marker and the Bouncer's seam that fills it go.
Its fix row runs the `landing` call through `verifytree.Verify` (D8).
Every `loom.yaml` key named `webster_*` for this perch becomes `code_*` with the same default: reviewer and fixer model, reviewer and fixer fan, judge, seed, `max_bounces`, `hard_max_bounces` and `auto_accept` (D14).
A Code-Perch re-entered after a route back follows the Bouncer's existing re-entry over a settled verdict (`internal/shedadapters/doc.go`): an unchanged task diff re-settles with no spawn, and a changed one runs an amendment generation scoped to the change, under `review_amend_max_bounces`.
The amendment's PR-Gate reject review input goes.

Rationale: the board puts a general perch after the merge, in place of the Webster perch.
The merge fixer's code is reviewed by it, which closes the gap the audit found in code a fixer wrote and nothing reviewed.

### D6. The Merge-Review perch and its keys go

Decision: the Merge-Review rows, `loom-rubric-merge-review`, the `merge-overlap` skip seam and the `merge` approve seam go.
The keys `merge_reviewer_model`, `merge_fixer_model`, `merge_reviewer_fan`, `merge_fixer_fan`, `merge_max_bounces`, `merge_hard_max_bounces` and `merge_auto_accept` go (D14).
The overlap computation the skip seam used stays, for D10.

Rationale: the board removes Merge-Review.

### D7. PR-Review: a stub row behind a flag per recipe

Decision: a new row `PR-Review`, between Describe and Finalize in both recipes, is a switched producer of `internal/shedadapters`.
It reads `loom.yaml`'s new `pr_review` in a loom run and `darn.yaml`'s new `pr_review` in a darn run, each default `false`.
Off, it is Done without acting; on, it is Done after one Info log that the PR review is not built.
It opens no pull request and calls no GitHub API either way.

Rationale: the board asks for the block now as a stub, behind separate loom and darn flags; the switched producer is the existing way a row reads its flag and skips.

### D8. Every gate after Webster is the `landing` gate

Decision: Darn-Write, Darn-Burler-Fix and Code-Burler-Fix run the `landing` call through `verifytree.Verify`, where they ran the `round` call; a darn diff wholly under `tree_scope` keeps its tree-scoped command.
With Publish gone, the `publish` gate kind has no site and goes from the gate schema (D14).
The `round` gate kind stays, since Webster's verify-gate fixer fork runs its call (`internal/websterengine/doc.go`), and Webster's gate is out of scope.

Rationale: the board makes `landing` the one gate after Webster, run by Merge-In, Finalize and the darn writer, so the darn writer ends only when every test passes.
A perch fix then reaches Finalize already verified under every tier, so Finalize's gate skips the tree and the lock stays short.

### D9. Darn's tail

Decision: the darn recipe is `Preflight` → `Darn-Write` → `Darn-Bouncer` ring → `Describe` → `PR-Review` → `Finalize` → `Friction-Reflect`, with no Merge-In.
Finalize's `on_reroute` names `Darn-Bouncer`.
Darn-Write loses its `PR-Review` segment, its `on_reroute` edge, its reroute on a failure record and its pending-rejection read.
`internal/loomrecipe`'s budget-0 rewrite routes Darn-Write's Done to `PR-Review` and Finalize's `on_reroute` to `PR-Review`, so neither the perch nor Describe ever runs and the writer's own description stands.

Rationale: the board's darn tail; the rewrite target moves with Merge-In's removal and stays a rewrite until `gate-runs-once` replaces it.

### D10. Finalize: one locked block, with a route back

Decision: Finalize is one producer whose block runs under the parent lock, taken first:

1. Take the lock, run the origin reconcile and the owner checks as today, and fast-forward a local parent behind its upstream.
2. Read the merge record's merged tip, then merge the parent in.
   On a conflict, a loom Finalize aborts the merge, releases the lock and reroutes, leaving the merge and its conflict session to Merge-In outside the lock; a darn Finalize resolves it with the conflict session under the lock, since darn has no other merging row.
3. Judge the merge large when it needed the conflict session or the parent's changes since the merged tip read in step 2 touch a file the task changed since its merge base; a darn run with no merge record measures from the task's merge base.
   Then write the merge record with the new merged tip, leaving the aligned tip alone.
   A large merge keeps the merge commit, releases the lock and reroutes, with no gate run and no fixer.
4. Run the `landing` gate through `verifytree.Verify`, which skips a tree it already verified.
   On a failure, spawn the fixer in `finalize` mode with `landing.yaml`'s new `finalize_fix_attempts`, default 2.
   A spent cap, or a fixer change whose tree diff from the pre-fix HEAD exceeds `landing.yaml`'s new `finalize_fix_max_lines`, default 40 added plus deleted lines, keeps the commits, releases the lock and reroutes.
5. Land: commit the parent records, squash with Describe's text and the trailer block as the commit message, with a `Lyx-Task: <slug>` line in the trailer block, mark the board task done and push the parent; no pull request is opened, looked up or closed.
6. Release the lock.

A reroute is a Stuck with Reroute on the row's `on_reroute` edge, never a goto, and spends no bounce budget.
Shed's reroute cap (`internal/shedengine/reroute.go`, three reroutes since the row's latest Done) bounds it: the next reroute halts the run at Finalize, and the Go driver hands that halt back (D11).
The lock is released on every way out of the block: Done, a reroute, every Stuck and every error; batten's idle-release backstop still frees a lock a dead Finalize left.
A parent moved outside the lock, by an operator push or on GitHub, is handled as today: the re-check after a merge-in-required error, and the parent push's rejection with its way forward.
The lock's longest hold is bounded by darn's conflict session (`conflict_timeout_min`) plus `finalize_fix_attempts` fixer turns of at most `fixer_timeout_min` each and their gate runs; a run that went right holds it for one merge, a skipped gate and the landing.

Rationale: the board's locked block and its open caps.
A conflict means the parent and the task changed the same lines, which a perch must see; in loom, Merge-In resolves it outside the lock.
Forty lines and two attempts admit the small fixes a run that went right needs; anything larger is reviewed on the way back.
Reusing the reroute edge and its cap adds no new loop guard.

### D11. Failures are the recipe's own

Decision: no verify failure at Merge-In or Finalize is routed to another row, and no driver resumes the landing halts on its own.
Removed: the failure record's target and row fields, `ClassifyVerifyFailure`, `VerifyFailureTarget` and the target's way-forward text; batten's publish retry (`publishretry.go`, its `<row>-publish-retry` marker and its settle-record check); the Go driver's verify-halt row and its `Verify` seam; `internal/arbiterstop`'s rule for a Merge-In or Publish verify halt; the round gate's read of the failure record (`internal/loomshed`'s `checkedPublishFailure` steps); Darn-Write's reroute (D9); and the settled-segment guard's exception for a verify failure.
Added: the Go driver hands back, ahead of every hand-over, arbiter and tackler row, a blocked stop whose reason starts with the reroute-cap prefix or the new landing-stop prefix that D4's two halts carry.
Batten's re-step of a halted Go child once per halt and per lyx build stays, so a fix deployed in a new build can still clear one.

Rationale: the board removes batten's retry and the route to Webster-Burler-Review; the fixer replaces both.
A tackler or arbiter resume would grant fresh fixer attempts or three more reroutes and spend another Opus fixer on an escalation no decision answered, so these halts go to the orch.

### D12. The landing failure record is the fixer's brief

Decision: the verifytree Publish failure record is renamed the landing failure record, in its type, accessors, file name and docs.
Merge-In and Finalize write it on a failed `landing` gate, with the gate, tier, failing tests, log, HEAD, merge-in commit and, in a loom run, the plan trace (`TraceFailure`).
A passing gate at either row removes it, and a later failure overwrites it.
The fixer's prompt names it, `Verify`'s log pruning keeps the log it names, and no other reader remains.
A record left by an older build under the old file name is ignored.

Rationale: the trace to plan cards, landed in `88b53f2a`, is the fixer's best head start; the old name names a row that no longer exists.

### D13. The PR and approval surface goes

Decision: removed with their tests, stencils and docs: Publish and its task-branch push, pull-request code and darn `PublishSkip` seam; PR-Align; PR-Gate; PR-Rework with its `rework-plan` gate, stencil and `rework` key; `lyx loom approve` and `lyx loom reject`; the alignment agent's `pr-gate` point; `alignment_pr_approve`; the PR-closing comment; and batten's parent-lock backstop arms for a PR-Gate hand-off and a moved parent at PR-Gate, keeping its idle release.
`landing.yaml`'s `require_pr_to_base` becomes `deploy_branches`, default `["main"]`, read only by batten's Land row to decide whether to deploy; an empty list deploys nothing.
`parent_lock_idle_release_min` loses its PR-Gate clause, and `skip_alignment`'s description names parent review alone.
`lyx loom review` and the parent-review gate stay.

Readers updated: `lyx report convergence` drops its PR-Rework, alignment-verdict and Merge-Review figures and reads Code-Review where it read Webster-Review.
The orch role stencil's PR-Gate section and its Merge-In/Publish verify-failure rule become one rule: a run halted at Merge-In has spent its fixer or escalated a decision for `lyx loom decision add`, and a reroute-cap halt at Finalize means a parent that keeps moving.
`CLAUDE.md`'s deploy paragraph says Batten's Land row deploys after every landing into a `deploy_branches` branch, and drops `alignment_pr_approve`.
`contracts/specs/loom-status-spec.md`, `docs/overview.md`, `README.md`, the recipe headers and each touched package's `doc.go` follow.

Rationale: the board's Gone list and its "what reads them goes with them or is updated".
`require_pr_to_base` would otherwise name a pull request no row opens while it decides only the deploy.

### D14. One migration for renamed and retired keys

Decision: one table in `internal/loomengine`'s config migration maps each renamed key to its successor and lists each retired key: `webster_*` perch keys to `code_*`, the `merge_*` perch keys, `alignment_pr_approve` and `rework` retired.
`lyx config reconcile` applies it to `loom.yaml` and to the hub's `profiles.yaml`, and `hubreconcile.Ensure` runs it after the binary change, so the hub needs no operator step.
The `run_settings` read in `internal/runprofile` and `configreg`'s profile check map a renamed key to its successor and drop a retired key with a Warn, never refusing it, so a board entry's old `run_settings` still seeds.
`landing.yaml`'s `require_pr_to_base` moves to `deploy_branches` keeping its value, an empty list included, and `gate.yaml`'s `publish` list is pruned.
`internal/configsync`'s former template values follow the same table.

Rationale: profiles and `run_settings` are checked against the module's keys at every write and at Seed-Child, so a rename without this table refuses every profiled start after landing.

## Constraints

- `PATTERN-told-geometry`: `internal/landingshed` keeps taking told paths, the fixer's included.
- `PATTERN-stencil-ownership`, `PATTERN-parent-directive`, `PATTERN-edit-directive`, `PATTERN-role-skills-typed`, `PATTERN-spawn-color`: the fixer's stencil is read at call time, renders both directives, and its launch spec names its skills and color segment.
- `PATTERN-shed-recipe-registry`: the PR-Review stub and the renamed rows register in `internal/shedrecipe`'s one map.
- `PATTERN-shed-driver`: the new hand-back row lives in `internal/sheddriver`.
- `PATTERN-refusal-way-forward`, `PATTERN-no-denied-recovery`: every new Stuck and halt names a way forward no agent's settings deny, with a test that reaches it.
- `PATTERN-tolerant-records`: the merge record and the landing failure record stay tolerant; dropped fields read as absent.
- `PATTERN-sole-parsers`: the fixer's report has one parser.
- `PATTERN-hub-wide-config`, `PATTERN-config-layers`, `PATTERN-template-overlay`, `PATTERN-binary-change-reconcile`: the new `landing.yaml` keys are hub-wide, and every rename and prune goes through D14's table.
- `PATTERN-origin-reconcile`, `PATTERN-owner-machine`: Merge-In and Finalize keep their reconcile first, and Finalize keeps its owner checks.
- `PATTERN-fabric-git`, `PATTERN-push-both-sides`, `PATTERN-push-failure-visible`: Finalize's landing push stays in `fabricengine`.
- `PATTERN-friction-capture`: every halt this task adds writes the loom halt note.
- `PATTERN-sandbox-coverage`, `PATTERN-cli-cobra`: removing `lyx loom approve` and `reject` updates the sandbox suite and the cobra tree.
- `PATTERN-verified-tree`: every gate run stays `verifytree.Verify`; its background file's sites change with D8, D12 and D13.
- Tests of removed rows and verbs go with them.

## Auto-mode assumptions

Every pick in D2 to D14 that the board left open is an auto-pick, recorded with its reasons in the support log's question ledger.

## Open risks

- Finalize's fixer changes code under the lock that no perch reviews; `finalize_fix_max_lines` bounds it, and a larger fix routes back to review.
- Merge-In's alignment runs on every merge that brings parent commits in, an Opus spawn in most loom runs; the audit's Merge-Review ran in 73% of runs at about $0.96 each.

## Acceptance criteria

- The loom recipe validates with D1's order and edges, the darn recipe with D9's, and the budget-0 darn recipe with D9's rewrite; the removed rows resolve to no producer.
- Merge-In runs to Done while another run holds the parent lock, merges the upstream tip, and leaves the local parent branch's ref unchanged.
- Merge-In over a local parent ahead of its upstream does not merge the local-only commit.
- With the local parent behind the merged tip, the Code-Perch subject, Describe's diff and the lints hold no parent commit.
- Merge-In with no parent commit brought in and a passing gate spawns no fixer; with parent commits brought in it spawns the fixer in `merge-in` mode even when the gate passes, the first Merge-In of a run included.
- Merge-In with a failing gate re-prompts the fixer until the gate passes or `merge_fix_attempts` is spent, then halts with a way forward naming the log.
- An `escalate` verdict halts Merge-In naming the report; the resumed Merge-In runs the fixer again.
- The Go driver hands back the fixer-spent, escalation and reroute-cap halts with no tackler or arbiter spawn.
- Finalize takes the lock first, fast-forwards a parent behind its upstream, and releases the lock on Done, reroute, Stuck and error.
- Finalize over a verified tree runs no gate and lands: squash with Describe's text and a `Lyx-Task: <slug>` trailer, board done, parent push, no GitHub call.
- Finalize over an overlapping parent change reroutes with the merge commit kept and no gate run; a loom conflict aborts the merge and reroutes to Merge-In; darn reroutes to Darn-Bouncer.
- Finalize's fixer past `finalize_fix_max_lines` or past `finalize_fix_attempts` reroutes with its commits kept.
- A fourth Finalize reroute with no Done between halts the run at Finalize.
- PR-Review is Done without acting with its flag off and on, in both recipes, with no pull request opened.
- Darn-Write, Darn-Burler-Fix and Code-Burler-Fix run the `landing` call, or darn's tree-scoped command.
- Batten sends a notice for a run halted at Merge-In or Finalize, with no goto.
- `lyx config reconcile` moves `webster_*` perch keys to `code_*` and `require_pr_to_base` to `deploy_branches`, an empty list kept empty, and prunes the `merge_*` perch keys, `alignment_pr_approve`, `rework` and the `publish` gate list, keeping the `round` list, in `loom.yaml`, `landing.yaml`, `gate.yaml` and `profiles.yaml`.
- A board entry whose `run_settings` hold `webster_max_bounces` or `merge_max_bounces` seeds, with the first read as `code_max_bounces` and the second dropped with a Warn.
- Batten's Land row deploys only for a parent in `deploy_branches`.
- `lyx loom approve` and `lyx loom reject` are unknown commands.
- `lyx report convergence` runs over runs recorded before and after the change.

## PATTERN changes

- `PATTERN-parent-lock` — changes: Moving a local parent branch, or merging into it and landing on it: only in Finalize under that parent's `internal/parentlock` lock; Merge-In merges the parent into a task branch without it, and a large merge or an over-cap fix at Finalize routes back through the recipe's reroute edge.
- `PATTERN-recipe-owns-failures` — adds: Failing a merge or test at Merge-In or Finalize: the row's own fixer fixes it or the run halts to the orch; no driver, batten, arbiter or record target routes it to another row.

## Notes for the plan writer

- `PATTERN-verified-tree`'s summary line stays; its background file's site list changes: Publish and the `publish` gate go, the `round` gate keeps only Webster's verify-gate fixer fork as its site, `Code-Burler-Fix`, Darn-Write and Darn-Burler-Fix run the `landing` call, the failure record is the landing failure record written by Merge-In and Finalize, and Merge-In's verify hold no longer runs under the parent lock.
- `loomengine.MigrateConfig`'s `retireSkipMergeReview` writes `merge_max_bounces: 0`; D14's prune runs after it.
- `internal/yamlengine/reconcile.go` treats `require_pr_to_base: []` as a meaningful empty list; `deploy_branches` keeps that treatment.
