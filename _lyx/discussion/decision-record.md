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
- `internal/fabricengine`'s merge-in: `MergeInAt`, a merge of a given parent SHA (D2).
- `internal/mergeresolve`'s `Resolve` signature and its `Fabric` seam, as an interface change only: the warp SHA D2 threads to `MergeInAt`.
- `internal/planparser`'s per-language test-file patterns, for the kind cap of Finalize's fixer and, at a Code-Review budget of 0, Merge-In's (D4, D10).
- The Webster-Review perch renamed and moved as the Code-Perch; the Merge-Review perch removed.
- The verify-failure routing removed from every reader (D11).
- The PR and approval surface removed (D13).
- Config: the `loom.yaml`, `darn.yaml`, `landing.yaml` and `gate.yaml` templates, the hub's `profiles.yaml` keys and board `run_settings` keys migrated at read time through each module's migration (D14).
- The landing halts' hand-back: `internal/sheddriver`'s row, `internal/arbiterstop`'s refusal and `internal/battenshed`'s re-step skip, the `hand_back` field on the stop envelope and `lyx shed status` in `internal/shedverbs`, and the `binary_changed` sites of the shed driver stencil and its repair guide (`shed-template-driver-guide.md`) (D11), and the Finalize check in batten's idle backstop and in `internal/loomcli`'s loop idle-release hook (D10).
- The Bouncer re-entry seam that feeds a re-entered perch the merge record's overlap or the merge fixer's report (D5, D10).
- Each module's migration, the gate module's new one included, and `configreg`'s Warn of its lines (D14).
- Readers updated in the same commit (D13).

Out:

- The parent-review alignment point before the Discussion perch: it stays, with `skip_alignment` and `alignment_timeout_min`.
- The conflict-resolution session (`internal/mergeresolve`) itself: its behavior is unchanged and only where it runs changes (D9), while its entry seam gains the warp SHA (D2).
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
Fabric's merge-in cannot do either today: `MergeIn` takes a branch name and resolves it at merge time, and `resolveMergeSources` with `pickMergeSourceSHA` (`internal/fabricengine/mergeguards.go`) merges the local SHA unless it is a strict ancestor of origin's, so a local parent ahead of its upstream is merged, and the tip read under the lock can move before the merge.
So fabric gains `MergeInAt(source, warpSHA string)`, which merges `warpSHA` on the code side in place of `resolveMergeSources`'s warp derivations from `source` and its freshness pick, and still names `source` for the records branch and the merge state's `Source`; `MergeIn(source)` stays as it is.
Merge-In reaches the merge through `mergeresolve.Resolve(ctx, source string)` and its `Fabric.MergeIn(source string)` seam (`internal/mergeresolve/deps.go`), so that path carries the SHA too: `Resolve` gains a warp SHA argument, empty for today's merge by name, and its `Fabric` seam gains `MergeInAt`, which `Resolve` calls when the SHA is set.
Merge-In passes the upstream tip it fetched or the local tip it read under the lock; Finalize passes no SHA, since it merges under the lock after its fast-forward (D10).
Merge-In takes the upstream tip from its own fetching read of the parent's tips, as today, never from the verb's fetch, which is best-effort and logs only a Warn; a failure of that read stays today's error stop.
A missing upstream ref and a mid-merge task pair stay Stuck with their way forward, as today.
Fast-forwarding the local parent branch and the parent-pair Stucks move into Finalize, under the lock (D10).
The conflict session resolves conflicts as today; Merge-In then runs the `landing` gate through `verifytree.Verify` and the merge fixer (D4), and writes the merge record with the tip it merged.

Rationale: the board removes the lock from Merge-In.
The fabric merge-in writes only the task pair's code side, since the weft is never a merge participant (`internal/fabricengine/doc.go`, "The merge surface"), so with the fast-forward gone Merge-In writes nothing parent-side.
The upstream tip is the parent as landed and pushed; the local tip can hold another run's squash whose push later fails and is reset (`internal/landingshed/doc.go`'s repair note).

### D3. The merge base every reader takes

Decision: the local parent branch can now lag the tip a task merged, so every reader of "the task's diff" or "since the merge base" takes the merge base of HEAD with the merge record's merged tip, falling back to the local parent branch only when no merge record exists.
The readers are the Code-Perch's review range and subject seam, Describe's diff, and the headroom and comment lints.

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
An absent aligned tip, as on a run's first Merge-In, reads as the task's fork point from the parent, never as a merge base taken from the merge record's merged tip.
In `merge-in` mode the fixer reads what the parent brought in since the aligned tip against the decision record, fixes what follows from the record, fixes the failing tests, and escalates a conflict between the parent's design and a decision of the record instead of choosing one.
Its report names each parent change it judged against the record and its finding on it.
A `fixed` verdict over a passing gate advances the aligned tip to the merged tip, and the row is Done; a merge that brought no parent commit and a passing gate spawn nothing, set the aligned tip to the merged tip and leave the row Done.
Merge-In writes the aligned tip on every Done, so no later Merge-In reads those parent commits again: the advance skips them on the fixer's word, bounded by its report, which the Code-Perch reads (D5).
Only Merge-In writes the aligned tip, and only in the merge record, whose reviewed-tip field becomes it; the reviewed-tip file, `MergeReviewedPath` and `RecordMergeReviewed` go with the Merge-Review perch.
An `escalate` verdict halts the run, its reason naming the report and the conflict, with the way forward `lyx loom decision add` in the task worktree to settle it, then `lyx loom resume`; the aligned tip stays, so the resumed Merge-In runs the fixer again over the amended record.
Merge-In's fixer gets `landing.yaml`'s new `merge_fix_attempts`, default 2, the fixer turns one row entry may spend, the same unit as D10's `finalize_fix_attempts`;
an `escalate` verdict spends no attempt.
A spent cap halts the run naming the failure record's log, with the way forward to fix on the task branch, then `lyx loom resume`;
the halt is noted in the row's attempt record, and the resumed Merge-In, which has no attempts left, reads the operator's fix as the alignment: over a passing gate it spawns no fixer, advances the aligned tip to the merged tip and is Done, and over a failing gate it halts again naming the log.
When the Code-Review perch's effective budget is 0, so that `zero_rounds` settles it approved at once and no review sees the `merge-in` fixer's change, D10's size and kind caps, with D10's language rule, bind that change too, measured as the tree diff from the entry's fixer baseline: the HEAD after the merge and the conflict session have committed, before the fixer's first turn.
Merge-In writes that baseline into the entry's attempt record when it first spawns the fixer, so a resume after an error stop, a failed spawn included, measures from the same HEAD, and the cap judges only the fixer's own commits, never the parent's merged commits or the conflict resolution.
Only a resume after the over-cap halt below, which the operator answered by reviewing the fixer's commits, writes the baseline afresh at the HEAD it resumes from; every other resume keeps the entry's baseline, so fixer commits made before an `escalate` or another halt still count, and an operator's commits made then count with them, over-covering rather than letting a fixer change escape.
A change over the caps halts the run with the landing-stop prefix, naming the fixer's commits, with the way forward to review and fix on the task branch, then `lyx loom resume`.
The halt is noted in the row's attempt record, and the resumed Merge-In reads the operator's review as the alignment: over a passing gate it spawns no fixer, advances the aligned tip to the merged tip and is Done, and over a failing gate it runs the fixer on the attempts the entry has left.
Every other fixer failure exit, at Merge-In and at Finalize, is a Stuck with Halt: a turn past `fixer_timeout_min`, a missing or unparseable report and a dirty tree after the turn name the session's log and report path, with the way forward to fix or clean the task branch, then `lyx loom resume`;
a pushed task branch names the pushed ref, with the way forward to check the pushed commits are the task's, then `lyx loom resume`, since the landing squashes the local task branch and never reads its remote.
A failed spawn spends no attempt and is an error stop with its transient class instead, which the arbiter or batten's re-step may resume;
the row keeps the attempts its entry spent in a record in the run's scratch directory, so no resume grants a fresh cap.
That attempt record, with the fixer baseline and the halt note it holds, belongs to one row entry: a resume continues the entry and reads it, while a re-entry of Merge-In or Finalize from another row, such as Merge-In after a Finalize reroute or Finalize entered again after its own reroute left a spent record, starts at zero spent attempts.
The record names the row entry it belongs to, so a resume finds its own entry's record and a re-entry finds an earlier one.
The row clears the record at its Done and at such a re-entry; a Finalize reroute leaves its record, which its next entry clears.
These halts and those above carry the landing-stop reason prefix that D11's driver row hands back.

Rationale: the board asks for one Opus agent, no twill, that reads what the merge brought in against the decision record.
One session re-prompted per failure keeps its context, as the row gates of Describe and the writers do.
The aligned tip, not the last merge, decides what the fixer reads, so a merge Finalize made before routing back (D10) is aligned at the next Merge-In.
Two attempts with an Opus-high session replace three Sonnet rounds, in line with the operator's view in `verify-fix-escalation`.
`landing.yaml` holds the keys because darn and loom share Finalize, and landing config is hub-wide.

### D5. The Code-Perch replaces the Webster-Review perch, after Merge-In

Decision: the Webster-Review perch is renamed and moved: segment `Code-Review`, rows `Code-Bouncer`, `Code-Burler-Review` and `Code-Burler-Fix`, `role_prefix` and `run_subdir` `code`, carry-over `Code-Review`.
Its subject is the task's committed diff at the merge base D3 names.
Its answer key is the decision record and the plan, and its review input adds the merge fixer's latest `merge-in` report, so the perch judges the alignment as well as the code; its rubric, renamed `loom-rubric-code-review`, reviews the code in general and against both, and its review range loses the PR-Gate rework heads and the failure-record merge commit.
The rubric's `publish_failure` marker and the Bouncer's seam that fills it go.
Its fix row runs the `landing` call through `verifytree.Verify` (D8).
Every `loom.yaml` key named `webster_*` for this perch becomes `code_*` with the same default: reviewer and fixer model, reviewer and fixer fan, judge, seed, `max_bounces`, `hard_max_bounces` and `auto_accept` (D14).
A Code-Perch re-entered after a route back follows the Bouncer's existing re-entry over a settled verdict (`internal/shedadapters/doc.go`): an unchanged task diff re-settles with no spawn, and a changed one runs an amendment generation scoped to the change, under `review_amend_max_bounces`.
A merge fixer report newer than the settled verdict also runs an amendment generation, over the report, even when the task diff is unchanged, through the re-entry seam D10's darn reroute uses.
The amendment's PR-Gate reject review input goes.

Rationale: the board puts a general perch after the merge, in place of the Webster perch.
The merge fixer's code is reviewed by it, which closes the gap the audit found in code a fixer wrote and nothing reviewed.

### D6. The Merge-Review perch and its keys go

Decision: the Merge-Review rows, `loom-rubric-merge-review`, the `merge-overlap` skip seam and the `merge` approve seam go.
The keys `merge_reviewer_model`, `merge_fixer_model`, `merge_reviewer_fan`, `merge_fixer_fan`, `merge_max_bounces`, `merge_hard_max_bounces` and `merge_auto_accept` go (D14).
The overlap computation the skip seam used stays, for D10.

Rationale: the board removes Merge-Review.

### D7. PR-Review: a stub row behind a flag per recipe

Decision: a new row `PR-Review`, between Describe and Finalize in both recipes, runs a new `PRReview` engine in `internal/landingshed`, registered in `internal/shedrecipe`'s one map.
The row's config names its `pr_review` setting, read through `Env.Setting`: `loom.yaml`'s new `pr_review` in the loom recipe and `darn.yaml`'s new `pr_review` in the darn recipe, each default `false`.
Off, it is Done without acting; on, it is Done after one Info log that the PR review is not built.
It opens no pull request and calls no GitHub API either way.
`PATTERN-command-row` was weighed against it and does not apply: a `Command` row over `pr_review` would behave as the stub does, since `CommandProducer` (`internal/shedadapters/command.go`) is Done with an Info log when its switch is off and when its command is empty, but the row is no side action.
It is the seat the human PR review engine will replace (Scope, Out), which gates the landing, and it must never take an operator shell command, which a `Command` row's command key would let an operator set.

Rationale: the board asks for the block now as a stub, behind separate loom and darn flags.
A `PRReview` engine gives the later review engine its row with no command key to retire; `landingshed` already holds the tail's other landing rows.

### D8. Every gate after Webster is the `landing` gate

Decision: Darn-Write, Darn-Burler-Fix and Code-Burler-Fix run the `landing` call through `verifytree.Verify`, where they ran the `round` call; at these darn rows and at a darn Finalize (D10), a darn diff wholly under `tree_scope` keeps its tree-scoped command.
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
   A parent pair missing, dirty or mid-merge is Stuck with its way forward, as at today's Merge-In, before any merge, and so, with no push skip, is one ahead of or diverged from its upstream.
2. Read the merge record's merged tip, then merge the parent in.
   On a conflict, a loom Finalize aborts the merge, releases the lock and reroutes, leaving the merge and its conflict session to Merge-In outside the lock; a darn Finalize resolves it with the conflict session under the lock, since darn has no other merging row.
3. Judge the merge large when it needed the conflict session or the parent's changes since the merged tip read in step 2 touch a file the task changed;
   the task side is the task's changes from its merge base to the task head before step 2's merge, as the Merge-Review skip seam's overlap computation takes them today, and a darn run with no merge record measures the parent side from the task's merge base.
   Then write the merge record with the new merged tip, leaving the aligned tip alone.
   A large merge keeps the merge commit, releases the lock and reroutes, with no gate run and no fixer.
   A darn large-merge reroute with a review budget above 0 hands Darn-Bouncer, through a Bouncer re-entry seam fed from the merge record, the parent's changes to the overlapping files since the merged tip step 2 read, or since the task's merge base where no merge record exists, as review input, and the re-entered perch runs an amendment generation over them even when the task diff is unchanged.
   In the budget-0 darn recipe the reroute reaches the PR-Review stub, so the operator's zero budget leaves the large merge to the next Finalize pass's gate alone.
4. Run the `landing` gate through `verifytree.Verify`, which skips a tree it already verified; a darn Finalize over a diff wholly under `tree_scope` runs the tree-scoped command in place of the `landing` call, as D8 states for the darn writer and fix rows.
   On a failure, spawn the fixer in `finalize` mode with `landing.yaml`'s new `finalize_fix_attempts`, default 2.
   A spent cap, or a fixer change over the cap, keeps the commits, releases the lock and reroutes.
   A fixer change is over the cap when its tree diff from the pre-fix HEAD exceeds `landing.yaml`'s new `finalize_fix_max_lines`, default 40 added plus deleted lines, or, whatever its line count, when it touches a test file of the run's language, any `testdata`, gate or build configuration, or adds a test skip or a build constraint.
   lyx holds no hub language setting (`internal/languageconfig` holds only skills and rename tools), so the cap, at Finalize and where D4 applies it at Merge-In, reads the language as follows:
   a loom run takes its live plan's `language:`, with an absent key read as `go`, as `effectiveLanguage` reads it;
   a darn run, which has no plan, takes the union of every declared language's patterns;
   and `none`, or a language the table lacks, treats every changed file as a test file, so the cap over-covers.
   The per-language test-file patterns are declared beside `internal/planparser`'s per-language `sourceExtensions` table (`reviewclass.go`), `_test.go` for Go, and for C# a `.cs` file whose name ends in `Tests.cs` or `Test.cs` or that lies under a directory whose name ends in `Tests`.
   They sit beside that table without sharing its default: there a language the table lacks declares no source file, while here it makes every file a test file.
   Where the route back reaches no review, a darn Finalize whose `on_reroute` is the budget-0 rewrite's `PR-Review` or a loom run whose Code-Review perch's effective budget is 0, its own `code_max_bounces` or the `review_max_bounces` it falls back to when `code_max_bounces` is null, a spent cap or an over-cap change halts with the landing-stop prefix instead of rerouting, naming the fixer's commits, with the way forward to review and fix on the task branch, then `lyx loom resume`.
   A `finalize`-mode `escalate` verdict keeps the fixer's commits, releases the lock and halts with the landing-stop prefix, naming the report, with the way forward `lyx loom decision add` in the task worktree, then `lyx loom resume`.
5. Land: commit the parent records, squash with Describe's text and the trailer block as the commit message, with a `Lyx-Task: <slug>` line in the trailer block, mark the board task done and push the parent; no pull request is opened, looked up or closed.
6. Release the lock.

A reroute is a Stuck with Reroute on the row's `on_reroute` edge, never a goto, and spends no bounce budget.
Shed's reroute cap (`internal/shedengine/reroute.go`, three reroutes since the row's latest Done) bounds it: the next reroute halts the run at Finalize, and the Go driver hands that halt back (D11).
The lock is released on every way out of the block: Done, a reroute, every Stuck and every error.
A lock a dead Finalize left is taken over at the next acquire, since `internal/parentlock` grants a record whose owner process no longer lives.
No idle release frees the parent lock while its owner's run is at Finalize and running, or while a verify marker for the run is live.
The check stands in both idle-release paths, beside each one's idle rule: the loop's own wake hook (`internal/loomcli/looplock.go`'s `loopIdleRelease` and `idleReleaseOn`) and batten's backstop (`internal/battenshed`'s `loopLockStep`).
Two more batten arms release the record: the owner-pause arm of `lockStep` and `ownedElsewhere`, each unless `keepsLockForFinalize` (`internal/battenshed/innerrun_owner.go`) holds.
That function reads `FinalizePastMerge`, true only once the parent holds the task's work, so these arms release the lock of a Finalize that is running before its landing.
They keep doing so, and `FinalizePastMerge` is unchanged: they act only while an owner pause or another machine's ownership stands, and a run another machine owns must not land from this one.
Finalize re-checks the lock before landing: immediately before step 5 it re-reads the parent lock record through its parent guard.
With the idle paths guarded, a record that no longer names this run means something released it, an owner pause, another machine's ownership or an operator's `lyx batten lock release`, so Finalize lands nothing and takes no lock: it is Stuck with Halt, naming the release and its three possible causes, with the way forward `lyx loom resume` once no owner pause stands and the run is this machine's, which after an operator's release holds already, so the resume needs no wait there.
The resume re-runs Finalize from step 1 and its owner checks under a lock taken afresh, so a resume made before the cause clears lands nothing: a run another machine owns stops at step 1's owner check, and a lock the owner-pause arm releases again is caught by this same re-check.
Finalize does not tell the causes apart: an owner pause is a live pause record, which step 1's owner checks do not read, so the halt names all three rather than guess one.
The re-check stands behind every release the idle paths do not guard, so no release path needs a guard of its own.
An unreadable record is a Stuck with Halt, as at the guard's acquire.
The re-read and the landing are not atomic: a release between them is caught, as today, only by the parent push's rejection.
A parent moved outside the lock, by an operator push or on GitHub, is handled as today: the re-check after a merge-in-required error, and the parent push's rejection with its way forward.
The retry pass after a merge-in-required error re-runs steps 2 to 4 under the same lock, so the lock's longest hold is bounded by two passes of darn's conflict session (`conflict_timeout_min`) plus `finalize_fix_attempts` fixer turns of at most `fixer_timeout_min` each and their gate runs; a run that went right holds it for one merge, a skipped gate and the landing.
The size and kind caps of step 4 are the bound on what an unreviewed Finalize fix can let through: a fix within them lands with no perch, and only the `landing` gate it passed and the `finalize` mode's instruction to fix failing tests only stand behind it.

Rationale: the board's locked block and its open caps.
A conflict means the parent and the task changed the same lines, which a perch must see; in loom, Merge-In resolves it outside the lock.
Forty lines and two attempts admit the small fixes a run that went right needs; anything larger, or any test or gate edit, is reviewed on the way back, except where the way back holds no review, where it halts to the orch.
Reusing the reroute edge and its cap adds no new loop guard.

### D11. Failures are the recipe's own

Decision: no verify failure at Merge-In or Finalize is routed to another row, and no driver resumes the landing halts on its own.
Removed: the failure record's target and row fields, `ClassifyVerifyFailure`, `VerifyFailureTarget` and the target's way-forward text; batten's publish retry (`publishretry.go`, its `<row>-publish-retry` marker and its settle-record check); the Go driver's verify-halt row and its `Verify` seam; `internal/arbiterstop`'s rule for a Merge-In or Publish verify halt; the round gate's read of the failure record (`internal/loomshed`'s `checkedPublishFailure` steps); Darn-Write's reroute (D9); and the settled-segment guard's exception for a verify failure.
The landing halts are the blocked stops at Merge-In or Finalize whose reason starts with the new landing-stop prefix that the halts of D4 and D10 carry, and the blocked stops at Finalize, the only landing row with a reroute edge (D1), whose reason starts with the reroute-cap prefix.
The reroute-cap prefix is written by shed for every row (`internal/shedengine/reroute.go`), so the producer match keeps Webster's plan-conflict reroute-cap halt out: it is routed as today.
The reroute-cap halt's way-forward clause, the told `Shed.RerouteCapWayForward` that `loomRerouteCapWayForward` (`internal/loomcli/wiring.go`) sets, names today only a parent-branch reroute and a Webster plan conflict.
It gains Finalize's causes that are not a parent settling: a fixer whose attempts are spent, with the way forward to fix the failing tests on the task branch, then `lyx loom resume`, and a fixer whose change is over the caps, with the way forward to review the fixer's commits and fix on the task branch, then `lyx loom resume`, as D10's other over-cap halts say.
A test reaches that clause.
Added: the Go driver hands back the landing halts, ahead of every hand-over, arbiter and tackler row.
`arbiterstop.Eligibility` refuses the landing halts, its refusal ending with `HandBackWayForward`, so the arbiter verb, the Go driver's escalation fact and batten's eligible-stop hold all hand them to the orch and batten's notice is not held for an arbiter.
Batten's binary re-step of a halted Go child, once per halt and per lyx build, skips the landing halts, a check in `internal/battenshed`'s re-step eligibility beside its blocked-or-failed test that reads the `hand_back` predicate below.
An LLM driver cannot name a row: the shed driver stencil (`contracts/stencils/shed/shed-template-driver.md`) is recipe-blind and branches only on envelope fields, policy words and error kinds.
So Go marks the landing halts in what the driver reads: a `hand_back` field on the stop envelope and on `lyx shed status`, set by a hand-back check on `internal/shedverbs`' Spec, told by the recipe's owner as `GotoGuard` is, so `shedverbs` stays recipe-generic.
That check is the producer-and-prefix test, and the Go driver's hand-back row and batten's re-step eligibility read the same predicate, so all three agree.
Every site that re-steps on `binary_changed` skips a stop marked `hand_back` and hands it back: the stencil's Re-launching bullet on `last_step.binary_changed`, its Parking step 3, which starts the binary watch for a binary-change-eligible stop, and the Binary watch section of `contracts/stencils/shed/shed-template-driver-guide.md`.
So an LLM-driven run neither starts the watch for a landing halt nor re-steps it after a build change.
So only the orch's `lyx loom resume` lifts the landing halts; every other halt keeps the re-step, so a fix deployed in a new build can still clear it.

Rationale: the board removes batten's retry and the route to Webster-Burler-Review; the fixer replaces both.
A tackler or arbiter resume would grant fresh fixer attempts or three more reroutes and spend another Opus fixer on an escalation no decision answered, so these halts go to the orch.

### D12. The landing failure record is the fixer's brief

Decision: the verifytree Publish failure record is renamed the landing failure record, in its type, accessors, file name and docs.
Merge-In and Finalize write it on a failed `landing` gate, with the gate, tier, failing tests, log, HEAD, merge-in commit and, in a loom run, the plan trace (`TraceFailure`).
A passing gate at either row removes it, as does a skipped gate over a tree `verifytree` records as passed, and a later failure overwrites it.
The fixer's prompt names it, `Verify`'s log pruning keeps the log it names, and no other reader remains.
A record left by an older build under the old file name is ignored.

Rationale: the trace to plan cards, landed in `88b53f2a`, is the fixer's best head start; the old name names a row that no longer exists.

### D13. The PR and approval surface goes

Decision: removed with their tests, stencils and docs: Publish and its task-branch push, pull-request code and darn `PublishSkip` seam; PR-Align; PR-Gate; PR-Rework with its `rework-plan` gate, stencil and `rework` key; `lyx loom approve` and `lyx loom reject`; the alignment agent's `pr-gate` point; `alignment_pr_approve`; the PR-closing comment; and batten's parent-lock backstop arms for a PR-Gate hand-off and a moved parent at PR-Gate, keeping its idle release.
`landing.yaml`'s `require_pr_to_base` becomes `deploy_branches`, default `["main"]`, read only by batten's Land row to decide whether to deploy; an empty list deploys nothing.
The deploy now runs every landing's code with the Code-Perch, or Darn-Review, as its last review where its perch budget is above 0, and Finalize's capped fix unreviewed.
A run whose perch budget is 0, such as one under the hub's `docs` profile at darn `review_max_bounces: 0`, deploys code with no review at all.
With the PR-Gate guard removed, such a run's deploy is bounded only by the landing gate, the operator's `deploy_branches` list and, over every fixer change it lands, D10's size and kind caps, Finalize's and, in a loom run, Merge-In's (D4).
The caps bound fixer changes alone: a conflict the conflict session resolved, at a darn Finalize under the lock and at a loom Merge-In, is no fixer change, so at a perch budget of 0 it lands with the landing gate alone, as D10 step 3 states for the budget-0 darn recipe.
`internal/landingshed`'s `doc.go` loses its claim that the deploy runs code PR-Gate reviewed.
`parent_lock_idle_release_min` loses its PR-Gate clause and gains D10's exception for a run at Finalize or with a live verify marker, and `skip_alignment`'s description names parent review alone.
`lyx loom review` and the parent-review gate stay.

Readers updated: `lyx report convergence` drops its PR-Rework, alignment-verdict and Merge-Review figures and reads Code-Review where it read Webster-Review.
The orch role stencil's PR-Gate section and its Merge-In/Publish verify-failure rule become one rule: a landing-stop halt at Merge-In or Finalize means the fixer spent its cap, failed, made a change no review on the way back can see, or escalated a decision for `lyx loom decision add`, and a reroute-cap halt at Finalize means a parent that keeps moving or a fixer that keeps failing to clear the gate within its caps, the cause read from the halt's reason.
`CLAUDE.md`'s deploy paragraph says Batten's Land row deploys after every landing into a `deploy_branches` branch, and drops `alignment_pr_approve`.
`contracts/specs/loom-status-spec.md`, `docs/overview.md`, `README.md`, the recipe headers and each touched package's `doc.go` follow.

Rationale: the board's Gone list and its "what reads them goes with them or is updated".
`require_pr_to_base` would otherwise name a pull request no row opens while it decides only the deploy.

### D14. Each module's migration carries its renamed and retired keys

Decision: each module's own config migration, the one `configreg` registers as its `Migrate`, maps each renamed key of that module to its successor and lists each retired key:
loom's `loomengine.MigrateConfig` moves the `webster_*` perch keys to `code_*` and retires the `merge_*` perch keys, `alignment_pr_approve` and `rework`, in one table that runs after `retiredReviewKeyRenames` and `copyWebsterModelsOntoMerge`;
landing's `landingshed.MigrateConfig` moves `require_pr_to_base` to `deploy_branches` keeping its value, an empty list included;
and the gate module gains a migration that prunes `gate.yaml`'s `publish` list.
`lyx config reconcile` applies them to the module files, and `hubreconcile.Ensure` runs them after the binary change, so the hub needs no operator step.
The hub's `profiles.yaml` stays seed-only and is never rewritten.
Profiles and `run_settings` are migrated at read time instead: `checkProfiles` and `LayerRunSettings` run each module's `Migrate` over every profile and `run_settings` document through `PrepareRunSettings` (`internal/configreg/runsettings.go`), so a renamed key reads as its successor and a retired key is dropped, never refused, and a board entry's old `run_settings` and a profile's old keys still seed.
`configreg.PrepareRunSettings` logs each line a module's `Migrate` returns at Warn, where it discards them today.
Since nothing rewrites a profile or a board entry, the Warn repeats at every read until the operator edits it; each Warn names the module and the key, with the way forward to rename or remove the key in `profiles.yaml` or in the entry's `run_settings`.
`internal/configsync`'s former template values follow the same tables.

Rationale: profiles and `run_settings` are checked against the module's keys at every write and at Seed-Child, so a rename without this table refuses every profiled start after landing.

### Added after Discussion (parent, 2026-10-23): Shorten the PATTERN-parent-lock text under the overview line cap

Decision: PATTERN-parent-lock's changes text is: Moving a local parent branch, or merging into it and landing on it: only in Finalize under that parent's `internal/parentlock` lock; Merge-In merges the parent into a task branch without it, and a loom conflict, a large merge or a spent or over-cap fix at Finalize reroutes, or halts where the reroute reaches no review.

Rationale: The declared text is 392 runes as an overview line, past the plan gate's 360-rune pattern-entry-line-cap; the shorter text says the same.

### Added after Discussion (parent, 2026-10-23): Shorten PATTERN-parent-lock under the headroom

Decision: Replace the declared PATTERN-parent-lock text with the shorter line below.

Rationale: PATTERN.md is past its headroom line, so the plan may not grow it; this set nets -13 bytes, and each cut clause moves to the entry's background file or the package docs the plan already edits.

### Added after Discussion (parent, 2026-10-23): Shorten PATTERN-recipe-owns-failures under the headroom

Decision: Replace the declared PATTERN-recipe-owns-failures text with the shorter line below.

Rationale: PATTERN.md is past its headroom line, so the plan may not grow it; this set nets -13 bytes, and each cut clause moves to the entry's background file or the package docs the plan already edits.

### Added after Discussion (parent, 2026-10-23): Shorten PATTERN-owner-machine for the headroom

Decision: Shorten the PATTERN-owner-machine overview line; the owner setter and machine-name rules move to pattern/PATTERN-owner-machine.md.

Rationale: PATTERN.md is past its headroom line, so the plan may not grow it; this set nets -13 bytes, and each cut clause moves to the entry's background file or the package docs the plan already edits.

### Added after Discussion (parent, 2026-10-23): Shorten PATTERN-pause-holds-prompts for the headroom

Decision: Shorten the PATTERN-pause-holds-prompts overview line, same meaning.

Rationale: PATTERN.md is past its headroom line, so the plan may not grow it; this set nets -13 bytes, and each cut clause moves to the entry's background file or the package docs the plan already edits.

### Added after Discussion (parent, 2026-10-23): Shorten PATTERN-model-choice-mapping for the headroom

Decision: Shorten the PATTERN-model-choice-mapping overview line, same meaning.

Rationale: PATTERN.md is past its headroom line, so the plan may not grow it; this set nets -13 bytes, and each cut clause moves to the entry's background file or the package docs the plan already edits.

### Added after Discussion (parent, 2026-10-23): Shorten PATTERN-failure-stop-visible for the headroom

Decision: Shorten the PATTERN-failure-stop-visible overview line, same meaning.

Rationale: PATTERN.md is past its headroom line, so the plan may not grow it; this set nets -13 bytes, and each cut clause moves to the entry's background file or the package docs the plan already edits.

## Constraints

- `PATTERN-told-geometry`: `internal/landingshed` keeps taking told paths, the fixer's included.
- `PATTERN-stencil-ownership`, `PATTERN-parent-directive`, `PATTERN-edit-directive`, `PATTERN-role-skills-typed`, `PATTERN-spawn-color`: the fixer's stencil is read at call time, renders both directives, and its launch spec names its skills and color segment.
- `PATTERN-shed-recipe-registry`: the PR-Review stub and the renamed rows register in `internal/shedrecipe`'s one map.
- `PATTERN-shed-driver`: the new hand-back row lives in `internal/sheddriver`.
- `PATTERN-refusal-way-forward`, `PATTERN-no-denied-recovery`: every new Stuck and halt names a way forward no agent's settings deny, with a test that reaches it.
- `PATTERN-tolerant-records`: the merge record and the landing failure record stay tolerant; dropped fields read as absent.
- `PATTERN-sole-parsers`: the fixer's report has one parser.
- `PATTERN-hub-wide-config`, `PATTERN-config-layers`, `PATTERN-template-overlay`, `PATTERN-binary-change-reconcile`: the new `landing.yaml` keys are hub-wide, and every rename and prune goes through D14's migrations.
- `PATTERN-origin-reconcile`, `PATTERN-owner-machine`: Merge-In and Finalize keep their reconcile first, and Finalize keeps its owner checks.
- `PATTERN-fabric-git`, `PATTERN-push-both-sides`, `PATTERN-push-failure-visible`: Finalize's landing push stays in `fabricengine`.
- `PATTERN-friction-capture`: every halt this task adds writes the loom halt note.
- `PATTERN-sandbox-coverage`, `PATTERN-cli-cobra`: removing `lyx loom approve` and `reject` updates the sandbox suite and the cobra tree.
- `PATTERN-verified-tree`: every gate run stays `verifytree.Verify`; its background file's sites change with D8, D12 and D13.
- Tests of removed rows and verbs go with them.

## Auto-mode assumptions

Every pick in D2 to D14 that the board left open is an auto-pick, recorded with its reasons in the support log's question ledger.

## Open risks

- Merge-In's alignment spawns an Opus session on every merge that brings parent commits in.
- At a Code-Review budget of 0 under a `language: none` plan, any `merge-in` fixer change counts as touching a test file, so such a run halts at Merge-In whenever its fixer changes a file (D4, D10).
- Finalize's lock re-check is not atomic with the landing: a release between the re-read and the squash lands as today, caught only by the parent push's rejection (D10).
- At a perch budget of 0, a conflict the conflict session resolved, at a darn Finalize or a loom Merge-In, lands with the landing gate alone, since it is no fixer change and no size or kind cap covers it (D10, D13).

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 4: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-4-review.md; fixer report: _lyx/reviews/discussion/round-4-fixer-report.md.
  - finalize-reroute-cap-halt-names-moving-parent-only (class consistency, severity MEDIUM, open in rounds 4)
  - merge-in-budget0-cap-baseline-precedes-merge (class design, severity MEDIUM, open in rounds 4)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- The loom recipe validates with D1's order and edges, the darn recipe with D9's, and the budget-0 darn recipe with D9's rewrite; the removed rows resolve to no producer.
- Over a parent with an upstream and no push skip, Merge-In runs to Done while another run holds the parent lock, merges the upstream tip, and leaves the local parent branch's ref unchanged.
- With the push skipped, Merge-In holds the parent lock only for its read of the local tip and has released it before its merge.
- Merge-In over a local parent ahead of its upstream does not merge the local-only commit, tested at fabric's `MergeInAt` and at Merge-In through `mergeresolve.Resolve`'s `Fabric` seam, where the merge carries the upstream tip's SHA.
- With the local parent behind the merged tip, the Code-Perch subject, Describe's diff and the lints hold no parent commit.
- Merge-In with no parent commit brought in and a passing gate spawns no fixer; with parent commits brought in it spawns the fixer in `merge-in` mode even when the gate passes, the first Merge-In of a run included.
- Merge-In with a failing gate re-prompts the fixer until the gate passes or `merge_fix_attempts` is spent, then halts with a way forward naming the log.
- An `escalate` verdict halts Merge-In naming the report; the resumed Merge-In runs the fixer again.
- In a loom run whose Code-Review perch's effective budget is 0, a `merge-in` fixer change past `finalize_fix_max_lines`, or one that edits a `_test.go` file under a Go plan, halts Merge-In with the landing-stop prefix naming its commits, while a merge whose parent changes and conflict resolution exceed the caps, followed by a fixer change within them, does not halt, and a resume after an error stop measures from the same baseline; at a budget above 0 the same change with a `fixed` verdict over a passing gate is Done and goes to the Code-Perch.
- The Go driver hands back the fixer-spent, escalation, fixer-timeout, budget-0 Merge-In over-cap and Finalize reroute-cap halts with no tackler or arbiter spawn, and the arbiter verb refuses each of them with `HandBackWayForward`; a Webster reroute-cap halt is not handed back by this row.
- A changed lyx build does not re-step a Finalize reroute-cap halt, a Merge-In escalation or a Merge-In fixer-spent halt, under the Go driver's batten re-step or the LLM driver's `binary_changed` re-launch; the stop envelope and `lyx shed status` mark each of them `hand_back`, and an LLM driver neither starts the binary watch for a marked halt nor re-steps it after the watch.
- Finalize takes the lock first, fast-forwards a parent behind its upstream, and releases the lock on Done, reroute, Stuck and error.
- A Finalize `landing` gate run longer than `parent_lock_idle_release_min`, with the fixer session waiting through it, keeps the lock, under both the loop's wake hook and batten's backstop.
- A Finalize whose lock record was released while its fixer waited halts before landing, with no squash and no push, and its resume re-runs Finalize from step 1 under a lock taken afresh; one whose run is then owned by another machine stops at the owner check without landing, and one whose record an operator's `lyx batten lock release` removed halts the same way and lands on `lyx loom resume` alone.
- Finalize over a verified tree runs no gate and lands: squash with Describe's text and a `Lyx-Task: <slug>` trailer, board done, parent push, no GitHub call.
- Finalize over an overlapping parent change reroutes with the merge commit kept and no gate run; a loom conflict aborts the merge and reroutes to Merge-In; darn reroutes to Darn-Bouncer, or to PR-Review in the budget-0 darn recipe.
- A darn overlap-only reroute runs a Darn-Review amendment generation whose input names the overlapping parent change.
- Finalize's fixer past `finalize_fix_max_lines` or past `finalize_fix_attempts` reroutes with its commits kept, and so does a fixer change under 40 lines that edits a `_test.go` file under a Go plan or a `FooTests.cs` file under a C# plan.
- A darn Finalize on a C# hub whose fix edits a `FooTests.cs` file reroutes with its commits kept, as does a loom Finalize under a `language: none` plan whose small fix edits any file.
- In the budget-0 darn recipe, Finalize's fixer past `finalize_fix_max_lines` halts with the landing-stop prefix instead of rerouting.
- A fourth Finalize reroute with no Done between halts the run at Finalize, and its way-forward clause names, beside the parent-branch cause, a spent fixer cap, to fix the failing tests on the task branch, and an over-cap fixer change, to review the fixer's commits and fix on the task branch, each then `lyx loom resume`.
- After a Finalize reroute on a spent `finalize_fix_attempts`, the Merge-In it reaches and the next Finalize entry each start at zero spent attempts and run their fixer; a Merge-In or Finalize resumed within one entry keeps the attempts that entry spent.
- PR-Review is Done without acting with its flag off and on, in both recipes, with no pull request opened.
- Darn-Write, Darn-Burler-Fix and Code-Burler-Fix run the `landing` call, or darn's tree-scoped command, and a darn Finalize over a diff wholly under `tree_scope` runs the tree-scoped command.
- Batten sends a notice for a run halted at Merge-In or Finalize, with no goto.
- `lyx config reconcile` moves `webster_*` perch keys to `code_*` and `require_pr_to_base` to `deploy_branches`, an empty list kept empty, and prunes the `merge_*` perch keys, `alignment_pr_approve`, `rework` and the `publish` gate list, keeping the `round` list, in `loom.yaml`, `landing.yaml` and `gate.yaml`, and leaves the hub's `profiles.yaml` unchanged.
- A profile holding `webster_max_bounces` and `merge_max_bounces` still validates and seeds, with the first read as `code_max_bounces` and the second dropped with a Warn.
- A board entry whose `run_settings` hold `webster_max_bounces` or `merge_max_bounces` seeds, with the first read as `code_max_bounces` and the second dropped with a Warn.
- Batten's Land row deploys only for a parent in `deploy_branches`.
- `lyx loom approve` and `lyx loom reject` are unknown commands.
- `lyx report convergence` runs over runs recorded before and after the change.

## PATTERN changes

- `PATTERN-parent-lock` — changes: Moving, merging into or landing on a local parent branch: only in Finalize, under its `internal/parentlock` lock. — added after Discussion (parent, 2026-10-23)
- `PATTERN-recipe-owns-failures` — adds: A merge or test failure at Merge-In or Finalize: its fixer, a Finalize reroute or a halt to the orch. — added after Discussion (parent, 2026-10-23)
- `PATTERN-owner-machine` — changes: Starting, resuming or landing a run or acting on wake: only where no other machine owns its entry. — [background](pattern/PATTERN-owner-machine.md) — added after Discussion (parent, 2026-10-23)
- `PATTERN-pause-holds-prompts` — changes: Typing into a covered session: shuttle refuses by slug; caller holds the line or claims the session. — [background](pattern/PATTERN-pause-holds-prompts.md) — added after Discussion (parent, 2026-10-23)
- `PATTERN-model-choice-mapping` — changes: Mapping a model-spec onto a spawn: only via `shuttleengine.NewModelChoice`. (test) — [background](pattern/PATTERN-model-choice-mapping.md) — added after Discussion (parent, 2026-10-23)
- `PATTERN-failure-stop-visible` — changes: An `--until-stop` failure as sole holder: status is non-running before the waiter returns. (test) — [background](pattern/PATTERN-failure-stop-visible.md) — added after Discussion (parent, 2026-10-23)

## Notes for the plan writer

- `PATTERN-verified-tree`'s summary line stays; its background file's site list changes: Publish and the `publish` gate go, the `round` gate keeps only Webster's verify-gate fixer fork as its site, `Code-Burler-Fix`, Darn-Write and Darn-Burler-Fix run the `landing` call, darn Finalize keeps the tree-scoped command for a tree-scoped diff, the failure record is the landing failure record written by Merge-In and Finalize, and Merge-In's verify hold no longer runs under the parent lock.
- `loomengine.MigrateConfig`'s `retireSkipMergeReview` writes `merge_max_bounces: 0`; D14's prune runs after it.
- `internal/yamlengine/reconcile.go` treats `require_pr_to_base: []` as a meaningful empty list; `deploy_branches` keeps that treatment.
