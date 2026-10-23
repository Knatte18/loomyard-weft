# Support log — loom-tail-review

## Interview

Autonomous run: no operator answered; every pick below is the writer's, recorded in the question ledger as an auto-pick.

### Turn 1 — scope and facts from the code

- Read the board entry, the related entries (`gate-runs-once`, `board-tidy-after-landing`, `speed-loom-profile`, `verify-fix-escalation`, `discussion-alignment-into-lenses`, `finalize-discrepancy-doc`, `run-verbs-shared`), both recipes, `internal/landingshed/doc.go`, the loom, darn and landing templates and the hub's `gate.yaml`.
- Facts settled: Describe already precedes Merge-In in loom; Webster's `final` gate already lists `[0, 1, 2]` on the hub, so board item 1 needs no code; shed's reroute cap is three reroutes since the row's latest Done (`internal/shedengine/reroute.go`); the fabric merge-in writes only the task pair's code side, so Merge-In's one parent-side write is the fast-forward; the Bouncer's re-entry over a settled verdict already runs an amendment generation scoped to a changed subject.
- Facts that shape removal: `require_pr_to_base` gates batten's deploy as well as Publish; the failure record feeds Darn-Write's reroute, the settled-segment guard, batten's retry, the Go driver's verify row and the round gate.

### Turn 2 — the fixer

- Q: is the merge fixer the conflict session, the alignment agent or a new session? Pick: a new landing session shared by Merge-In and Finalize, with the conflict session kept for conflicts.
- Q: when does Merge-In spawn it with a passing gate? First pick: when the parent's changes overlap the task's files; revised in turn 5 to every merge that brings parent commits in.
- Q: attempt cap and escalation? Pick: two gate failures per entry, one re-prompted session; an escalation halts and resumes through `lyx loom decision add`.

### Turn 3 — Finalize's caps and the large merge

- Q: what is a large merge? Pick: a conflict, or a parent change since the previous merged tip that touches a file the task changed.
- Q: caps? Pick: two fixer attempts and forty changed lines; past either, keep the commits and reroute; shed's reroute cap of three ends the loop.
- Q: does a large merge get tested before the route back? Pick: no; the next pass tests it.

### Turn 4 — names, keys and the PR stub

- Q: keep the Webster-Review names for the moved perch? Pick: rename to Code-Review and `code_*`, with reconcile moving set values.
- Q: what happens to `require_pr_to_base`? Pick: renamed `deploy_branches`, read only by batten's Land row.
- Q: the failure record? Pick: kept as the fixer's brief without a target, renamed the landing failure record.

### Turn 5 — lens review

All seven forks returned a report.

- bound-breaker, BLOCKING, local-ref merge base counts merged parent commits as the task's: fixed, new D3 names the merged tip as every reader's base.
- bound-breaker, BLOCKING, lock-free Merge-In can merge another run's unpushed squash: fixed, D2 merges the upstream tip, and the local tip only under a brief lock when there is no upstream.
- existing-paths, BLOCKING, profiles and `run_settings` refuse renamed keys at Seed-Child: fixed, new D14 with one migration table, reconcile over `profiles.yaml` and a tolerant `run_settings` read.
- goal-scope, BLOCKING, the darn writer's gate is not the `landing` gate: fixed, new D8.
- goal-scope, MEDIUM, Code-Burler-Fix keeps `round` against the board's one gate after Webster: fixed in D8, which also removes the `round` gate kind; this also answers interactions LOW 5 and walkthrough LOW 4 on tmux failures surfacing under the lock.
- goal-scope, MEDIUM, landing commit does not name the slug: fixed, `Lyx-Task: <slug>` trailer in D10.
- goal-scope, MEDIUM, alignment narrowed to overlapping changes: fixed, D4 aligns every merge that brings parent commits in; the old open risk is replaced by the cost risk, since the audit's Merge-Review ran in 73% of runs at about $0.96 each.
- walkthrough, MEDIUM, large-merge test reads the merged tip after it is overwritten: fixed, D10 step 2 reads it first and step 3 writes after the judgement.
- walkthrough, MEDIUM, interactions MEDIUM 2 and bound-breaker MEDIUM 4, the landing halts go to the tackler and arbiter under the Go driver and get fresh caps: fixed, D11 adds a hand-back row for the reroute-cap and landing-stop prefixes.
- walkthrough, MEDIUM, and bound-breaker MEDIUM 7, no aligned tip on the first Merge-In: fixed, it reads as the task's merge base; only Merge-In writes it (also interactions LOW 4).
- walkthrough, LOW, `run_settings` claim for the PR-Review flag unverified: fixed, the claim is dropped.
- interactions, MEDIUM, and bound-breaker MEDIUM 6, darn budget 0 reroutes into the perch and Describe: fixed, D9's rewrite also points Finalize's reroute at PR-Review.
- interactions, MEDIUM, and existing-paths MEDIUM, the round gate and arbiterstop still read the failure record: fixed, both reads go in D11 and D12 states no other reader.
- existing-paths, MEDIUM, arbiter eligibility: fixed with the hand-back row and the arbiterstop rule's removal.
- existing-paths, MEDIUM, the rubric's `publish_failure` marker: fixed, it goes in D5.
- existing-paths, MEDIUM, batten's PR-Gate lock backstop: fixed, D13 removes the PR-Gate arms and keeps the idle release, which D10 cites.
- existing-paths, LOW, old merge records carry a review tip: rejected, in-flight runs restart, so no old record reaches the new Merge-In.
- existing-paths, LOW, `configsync` former values and migration order: fixed, D14 and the notes.
- existing-paths, LOW, darn `PublishSkip` seam: fixed, listed in D13.
- bound-breaker, MEDIUM, "parent cannot move under the lock" is false: fixed, D10 now names today's re-check and push rejection for an outside move.
- bound-breaker, MEDIUM, conflict session under the lock: fixed for loom, whose Finalize aborts a conflict and reroutes; darn keeps it and D10 states the lock's bound.
- bound-breaker, LOW, fix-size measure ambiguous: fixed, tree diff from the pre-fix HEAD.
- bound-breaker, LOW, empty `require_pr_to_base` list: fixed, D14 keeps an empty list and the notes name reconcile's treatment.
- bound-breaker, LOW, rubric range steps for rework heads and the failure merge commit: fixed in D5.
- goal-scope, LOW, renames unlicensed and the deploy step: kept with reasons, and D14 states the migration needs no operator step.
- goal-scope, LOW, every conflict counts as large: kept as a deliberate departure, recorded in the ledger.
- goal-scope, LOW, Bolt orch rules stale between landing and rewrite: fixed, Scope makes the rewrite a step of landing.
- goal-scope, LOW, `skip_alignment` description: fixed in D13.
- board-fit, MEDIUM, `regression-baseline-verify` and `loom-beam-darns` built on the old tail: fixed, named in Scope's Out list for the orch to amend.
- board-fit, LOW, more entries naming the old tail and the overlap with `claude-md-lyx-rules-out`: fixed, named in Scope.
- board-fit, LOW, `discussion-alignment-into-lenses` premise flips: fixed by naming it for amendment; the amendment's content is the orch's.
- leanness, MEDIUM, fact-dependent Constraint on Merge-In's parent writes: fixed, cut.
- leanness, MEDIUM, `PATTERN-verified-tree` bullet repeats the entry: fixed, moved to the notes as a background change.
- leanness, LOW, loomcli file list in Notes: fixed, moved to `relevant.md`.
- leanness, LOW, Auto-mode assumptions restate decisions: fixed, one pointer line.
- leanness, LOW, parent-lock Constraint, landing open risk and generic test and doc constraints: fixed, cut or trimmed.
- leanness, LOW, Goal's row list repeats D1 and D9: rejected, the goal states the tail's shape in one place the plan writer reads first.

## Rejected alternatives

- The fixer as its own recipe row after Merge-In: Finalize needs the same fixer inside its locked block, where a row hop would release the lock.
- Folding conflict resolution into the fixer: the conflict session already works in both rows, and merging the two changes `internal/mergeresolve` for no gain the board asks for.
- Aligning only merges whose parent changes overlap the task's files: a parent's new PATTERN entry or changed contract can contradict a decision without touching a task file, and the board asks the fixer to read what the merge brought in.
- Keeping Webster-Review's names and keys: the perch no longer follows Webster or reviews against the plan alone, and the restart rule makes the rename free for runs.
- Testing a large merge under the lock before rerouting: the lock is meant to be short, and the rerouted pass tests the tree anyway.
- Resolving a loom conflict under Finalize's lock: the conflict session can run up to an hour, and Merge-In resolves outside the lock.
- A fix-size cap in files rather than lines: one file can hold a large rewrite.
- Removing the failure record altogether: the plan trace it carries is the fixer's best head start.
- Keeping `require_pr_to_base` for the deploy: its name would claim a pull request no row opens.
- Merging the local parent tip without a lock: it can carry another run's squash whose push fails and is reset.
- Taking the lock in Merge-In for the fast-forward: a lock wait behind another run's Finalize defeats the board's lock-free Merge-In.
- Leaving the landing halts to the tackler: a resume grants fresh caps and reroutes.

## Review rounds

### Parent review, round 1

- F1 (BLOCKING, scope): fixed. Removing the `round` gate kind would have reached Webster's verify-gate fixer fork, which runs the `round` call. D8 now removes only the `publish` kind and keeps `round`. D14, the reconcile acceptance criterion and the `PATTERN-verified-tree` note now prune only `gate.yaml`'s `publish` list. The turn-5 lens line on goal-scope's MEDIUM finding holds only for the move of the post-Webster fix gates to `landing`.

### Discussion review, round 2

- Every finding fixed in the decision record: the Finalize lock check in both idle-release paths (F1), the given-SHA merge (F2), the LLM driver's re-launch exclusion (F3), read-time profile migration (F4), per-language test files (F5), the budget-0 deploy bound (F6), the `PRReview` engine (F7), the reroute in `PATTERN-recipe-owns-failures` (F8), the reroute-cap match limited to Finalize (F9), darn Finalize's tree-scoped command (F10), fixer turns as one unit (F11) and the effective Code-Review budget (F12).

### Discussion review, round 3

- Every finding fixed in the decision record: batten's owner arms and Finalize's lock re-check and halt before landing (F1), `MergeInAt` threaded through `mergeresolve.Resolve` (F2), the `hand_back` field the LLM driver's `binary_changed` sites read (F3), the language Finalize's kind cap reads (F4), D10's caps on the Merge-In fixer at a Code-Review budget of 0 (F5) and D7's reason for the `PRReview` engine (F6).

### Discussion review, round 4

- Every finding fixed in the decision record: the budget-0 Merge-In cap measured from the fixer baseline after the merge (F1), the Finalize fixer causes in the reroute-cap way-forward clause and the orch stencil rule (F2), the conflict session's resolution in the budget-0 deploy bound (F3), the operator's lock release among the re-check's causes (F4) and the attempt record's reset per row entry (F5).

## Question ledger

- Fixer placement (session in Merge-In and Finalize vs a row) — resolved, auto-pick: session; see D4.
- Fixer model — resolved, auto-pick: `opus[high]` in `landing.yaml`'s `fixer`.
- Merge-In fixer trigger — resolved, auto-pick: a failed gate or any parent commit since the aligned tip; see D4.
- Merge-In attempt cap — resolved, auto-pick: `merge_fix_attempts` 2.
- Escalation path — resolved, auto-pick: halt handed back to the orch, `lyx loom decision add`, `lyx loom resume`.
- Large merge definition — resolved, auto-pick: conflict or overlap since the previous merged tip; see D10.
- Conflict at Finalize — resolved, auto-pick: counts as large, a deliberate stricter reading of the board's "on a conflict … fix it"; loom aborts and reroutes, darn resolves under the lock and reroutes.
- Finalize caps — resolved, auto-pick: `finalize_fix_attempts` 2, `finalize_fix_max_lines` 40 measured as a tree diff.
- Route-back cap — resolved, auto-pick: shed's existing reroute cap, handed back by the Go driver.
- Code-Perch naming — resolved, auto-pick: `Code-Review`, `code_*`.
- Gates after Webster — resolved, auto-pick: `landing` everywhere, the `publish` kind removed and the `round` kind kept for Webster's verify-gate fixer fork; see D8.
- PR-Review flag names and default — resolved, auto-pick: `pr_review`, false, in `loom.yaml` and `darn.yaml`.
- Slug in the landing commit — resolved, auto-pick: a `Lyx-Task: <slug>` trailer line.
- `require_pr_to_base` — resolved, auto-pick: renamed `deploy_branches`.
- Failure record — resolved, auto-pick: kept and renamed, no target, no reader but the fixer.
- Go driver verify row — resolved, auto-pick: removed with its seam, replaced by a hand-back row.
- Key migration — resolved, auto-pick: one table per module, applied by reconcile to the module files and at read time to profiles and `run_settings`, `profiles.yaml` staying seed-only; see D14.
- Merge-In's merge source — resolved, auto-pick: fabric's merge-in gains a given-SHA merge, since `MergeIn` by name merges a local tip ahead of its upstream; see D2.
- PR-Review engine — resolved, auto-pick: a new `PRReview` engine in `internal/landingshed`, weighed against a `Command` row under `PATTERN-command-row`, which would pass on as the stub does but would let an operator set a shell command on the seat the human PR review engine will replace; see D7.
- Hub orch rules naming the old tail — resolved: a landing step for the orch through `lyx orch rules`.
- Batten's owner arms under a running Finalize — resolved, auto-pick: they keep releasing the lock, since a run another machine owns must not land here, and Finalize re-checks the lock before landing and halts when the record no longer names it, rather than taking the lock again while the pause stands; see D10.
- Language of Finalize's kind cap — resolved, auto-pick: the loom plan's `language:`, the union of every declared language for darn, every file for `none` or an undeclared language; see D10.
- Merge-In fixer at a Code-Review budget of 0 — resolved, auto-pick: D10's size and kind caps bind it, halting past them; see D4.
- Baseline of the budget-0 Merge-In cap — resolved, auto-pick: the HEAD after the merge and conflict session, before the fixer's first turn, kept in the entry's attempt record and written afresh only after the over-cap halt the operator reviewed, so earlier fixer commits never escape the cap; see D4.
- Lifetime of the fixer attempt record — resolved, auto-pick: one row entry, naming its entry, kept across resumes and cleared at Done and at a re-entry from another row; see D4.
- Resume after Merge-In's spent cap — resolved, auto-pick: the operator's fix reads as the alignment, so a passing gate advances the aligned tip with no fixer, since the entry has no attempts left; see D4.
- Causes of a released Finalize lock — resolved, auto-pick: the halt names all three, owner pause, other machine and operator release, since step 1's owner checks do not read the pause record; see D10.
