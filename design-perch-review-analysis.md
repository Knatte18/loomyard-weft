# Perch review analysis: rounds, tokens and late BLOCKING findings, with proposals

`perch-review-analysis` · Note · enhancement, burler, loom, shed

Imported from [issue #676](https://github.com/Knatte18/loomyard/issues/676).

This issue collects an analysis of the review perches (Bouncer + Burler rounds) from the records in `loomyard-weft`, and the proposals that follow from it.
All numbers come from the newest `archive/<slug>/<sha>` tag per slug, read on 2026-10-16.
They cover review-side records only (`_lyx/reviews/<phase>/round-N-*.md` and `round-N-usage.yaml`); writer, implementer and orch sessions are not in weft and are not counted here.

## Context: throughput since LoomYard took over its own development

Main lands one commit per task in both eras (Millhouse squash merges before 2026-09-30, `lyx` landings after), so tasks per week compare directly (`manifest:` and `chore:` commits excluded).

| Period | Tasks landed per week | Go code lines changed per week | Go test lines changed per week |
|---|---|---|---|
| Millhouse, June–September | about 25 | about 15 000 | about 24 000 |
| LoomYard, week of 2026-09-28 | 92 | 46 000 | 111 000 |
| LoomYard, 2026-10-05 – 10-11 | 137 | 87 000 | 237 000 |
| LoomYard, 2026-10-12 – 10-16 (4 days) | 88 | 73 000 | 91 000 |

Tasks did not get smaller: the median task changed 64 → 216 Go code lines, 170 → 382 test lines and 12 → 18 files, and the share of tasks under 50 Go lines fell from 42 % to 29 %.

## Where review tokens go

Summed over 170 runs that have `round-N-usage.yaml` (fresh = input + output + cache write, including forks):

| Perch | Fresh tokens | Share | Cache read |
|---|---|---|---|
| Discussion (seed + review + fix + judge) | 75 M | 42 % | 960 M |
| Plan | 60 M | 34 % | 1 440 M |
| Webster | 33 M | 18 % | 570 M |
| Merge and darn | 10 M | 6 % | 100 M |
| Total | 179 M | | 3 080 M |

The reviewer session costs about two thirds of each perch; the fixer most of the rest.
Plan review reads the most from cache (1.1 B) and is the slowest (74 h review, 45 h fix in total), consistent with reviewers reading much of the tree to check a plan.
Each round starts fresh review, fix and judge sessions that re-read their context, which is why cache reads dominate.

## Rounds per perch

Over the 96 runs with usage records for all three perches:

| Perch | Mean | 1 round | 2 rounds | 3 rounds | 4 or more |
|---|---|---|---|---|---|
| Discussion | 2.5 | 4 % | 53 % | 35 % | 7 % |
| Plan | 1.8 | 41 % | 41 % | 14 % | 5 % |
| Webster | 1.6 | 75 % | 3 % | 12 % | 9 % |

Runs without a perch's usage file are left out: darn runs, `parent-review`-only runs, discussion-only runs, runs from before usage files were recorded, and a few with timestamped directories such as `plan-20261007T…`.

### Early look after the twill writers

25 comparable runs archived after `2abb06c5` (twill plan writer, 2026-10-15 08:35):

| | Before (96 runs) | After twill (25 runs) |
|---|---|---|
| Discussion, mean rounds | 2.5 | 2.4 |
| Discussion, approved in round 1 | 4 % | 16 % |
| Plan, mean rounds | 1.8 | 1.6 |
| Plan, approved in round 1 | 41 % | 56 % |
| Webster, approved in round 1 | 75 % | 96 % |

The sample is small, some of these runs started before twill landed, and the landing time of the twill discussion writer was not checked separately, so this is a direction, not a result.

## BLOCKING findings do not converge to zero

Share of review rounds with at least one BLOCKING finding:

| Perch | Round 1 | Round 2 | Round 3 | Round 4+ |
|---|---|---|---|---|
| Discussion | 51 % | 29 % | 24 % | 23 % |
| Plan | 42 % | 36 % | 38 % | 33 % |
| Webster | 21 % | 49 % | 37 % | 40 % |

The rate drops after round 1 and then flattens at 25–40 %, so a hard cap at 3 would cut real findings unless the late ones are noise.

### What the late BLOCKING findings are

Every BLOCKING finding in rounds 4 and later (31 findings across 15 runs) was classified against the artifact history and fixer reports:

| Perch | A: introduced by an earlier fix | B: there from the start, missed | C: not really blocking | Total |
|---|---|---|---|---|
| Discussion | 4 | 3 | 0 | 7 |
| Plan | 1 | 3 | 0 | 4 |
| Webster | 0 | 7 | 13 | 20 |

- **Discussion:** all late findings are real design holes, and most are consistency regressions from fixes.
  A fixer replaced one false claim with another (shuttle-state, round 3 → 4), or changed D6 without updating the sibling D7 it now contradicted (engine-model-choice).
- **Plan:** the holes were in the first draft, and each fix covered only part of the affected set.
  In config-layers-run-settings, round 4 found four stale clone tests under one key and round 5 found three more under the same key.
- **Webster:** 13 of 20 were comment or markdown line-break style rated BLOCKING, about 7 of them on lines a fixer had just re-wrapped at the reviewer's request, so the loop generated its own findings.
  The real behaviour bugs (gate-test-scope F1, reed-multi-window F1/F2) were there from the start, and the reed ones surfaced only when real tmux ran at publish, because earlier rounds ran fake tests only.
- **Across perches:** in most runs the round-4 fix was never reviewed again, because the escalation was accepted at the budget or hard cap.

Webster A vs B rests on fixer reports and review texts, not code diffs: the per-round warp commits are gone after the squash merges.
The full table is below.

## Already done

Prose line breaks and formatting are never a BLOCKING finding in Webster and the perches.

## Proposals

1. **Webster hard cap of 2.**
   With style no longer BLOCKING, most late Webster rounds disappear, and the remaining real bugs were caught by tests rather than review.
   Keep prose and comment style as a mechanical gate (`lyx loom lint-comments`), never as a reviewer finding.
2. **Two parallel reviewers in round 1, one fixer for both.**
   One reviewer per round behaves like a sample: the B findings above existed in round 1 and were found only later.
   Two independent full reviewers in round 1 should catch more of them up front, with less tuning than the fan-out lenses need.
3. **Later rounds review the fix delta.**
   A findings cannot be caught in round 1; they need a round after the fix.
   From round 2, one reviewer checks only what the fixer changed plus the sibling decisions or cards it touches, which is cheaper than a full review.
   Compare Traycer's incremental re-verify in #671.
4. **Fixer duties.**
   A discussion fixer that changes a decision re-checks its siblings for contradictions.
   A plan fixer enumerates every instance of the finding's key (for example through `lyx quarry`) instead of fixing only the instances the reviewer listed.
   Every new claim the fixer writes is checked against the code before it is written, including claims it takes over from the reviewer's proposed fix.
5. **A twill fixer: fix, then verify with forks of itself.**
   The five A findings fall in two groups.
   In two, the fixer trusted the reviewer: it wrote the reviewer's false premise into the record (config-layers-run-settings) or applied the reviewer's proposed way forward, which was itself a dead end (webster-dead-ends).
   In three, the fixer fixed locally without checking the consequences: D6 changed without D7 (engine-model-choice), one false bound replaced by another (shuttle-state), an added clause left unbounded (logging-cleanup).
   `burler-step-fix.md` already has the fixer weigh each finding's premise and proposed change and dispute them, and `docs/research/twill-fixer-bench.md` found that critical fixer safe but no better on final plans, so more critique before the fix is not the gap; checking after the fix is.
   Proposal: after the fixer writes its changes, it spawns forks of itself that each verify one aspect of what it just changed:
   - consistency: every sibling decision or card that the changed text touches still agrees with it;
   - claims: every sentence the fix added or changed that asserts a bound, a guarantee or a behaviour holds against the code;
   - way forward: every refusal or next step the fix introduced can be carried out from the state it names.
   The fixer folds the forks' results into its own fix before the next Bouncer round, so a fix-induced defect is caught inside the round that made it, without a fresh review session.
   The forks share the fixer's context, so they cost far less than a new review round.
6. **Measure again in a week**, with the same scripts: rounds per perch, BLOCKING share in round 1, and tokens per perch, before and after twill and the changes above.

Doubling reviewers in every round instead of only round 1 is the simpler variant, but it doubles the most expensive part of each perch.

<details>
<summary>Full classification of the late BLOCKING findings</summary>

## Discussion

| perch | round | slug | finding title | class | justification | bouncer verdict | resolved next round? |
|---|---|---|---|---|---|---|---|
| discussion | 4 | config-layers-run-settings | `loom.discussion_producer` opens a review switch-off to the local layer | A | The targeted sentence ("keeps `discussion_producer` open because it picks the writer, not a reviewer") was added by the round-3 fixer (round-3-fixer-report.md:12), following round-3 review's own claim (round-3-review.md:58) that the key acts on the writer. The key being open is part of the recurring LocalKeys-enumeration defect, but the false premise itself is a round-3 fix product. | CIRCLING | No round-5 review; round-4 fixer added the key to LocalDenied, escalation accepted (cause budget) and the parent appended a D4 addendum making it LocalDenied. |
| discussion | 4 | engine-model-choice | D7's wiring refusal contradicts D6's no-key-check rule | A | D7's "refused at wiring" sentence is in the initial record (633c2f347 line 108) and was never touched, but the D6 rule it contradicts (`record-batch`, `reset` etc. check no key; checks at spawn sites) was written by the round-2 and round-3 fixers (round-2-fixer-report.md:13, round-3-fixer-report.md:10). The inconsistency is a consistency regression from those D6 fixes not propagating to D7. | CONTINUE | No round-5 review; round-4 fixer made D7's wiring refuse nothing (round-4-fixer-report.md:6), then escalation accepted at hard cap. |
| discussion | 4 | logging-cleanup | D17's ready-task fallback starts any ready task, unbounded | A | The "starts any task the board shows ready and not yet started" clause was introduced by the round-2 fixer (round-2-fixer-report.md:5) and decoupled from the deploy-gated wake by the round-3 fixer (round-3-fixer-report.md:6), which made it unbounded. Not in the initial record. | CIRCLING | No round-5 review; round-4 fixer bounded it to the notice-log `newly ready` line, then escalation accepted at hard cap. |
| discussion | 4 | loom-bouncer-rounds | D4 budget-reached test rests on a budget the seam does not report | B | The core defect (D4 reads the "segment's" count through the `Bounces` seam while D3 drops the literal `max_bounces` so the seam's budget falls to `defaultMaxBounces`) is in the initial record (63b1e694a D3/D4 lines 62-73); rounds 1-3 flagged only the off-by-one, never the seam's budget source. The round-3 fix made the `review_max_bounces` comparison explicit, which sharpened but did not create the mismatch. | CIRCLING | Yes: round-4 fixer stated the test as the seam's `count >= budget` and required the seam's routing to go through the config assembly; round 5 CONVERGED and confirms the D4 budget test. |
| discussion | 4 | reed-view | Zoom does not survive reed's own resize and apply paths | B | D1's zoom (`resize-pane -Z`) and D7's `window-resized` pin hook are both in the first decision-record commit cb84bc254, and rounds 1-3 reviewed D7's pin arithmetic three times without checking the pins against zoom. Fixers 1-3 refined pin heights but did not create the conflict. | CONTINUE | Yes: fixer 4 added re-zoom around every geometry step plus a Bound and criterion; round 5 has only MEDIUM/LOW follow-ups on D1 (pane order while zoomed, other unzooming commands). |
| discussion | 4 | shuttle-state | D1 bound on line corruption is false for the rewritten Stop and ask hooks | A | The false sentence ("as today's pair can … adds no new way to corrupt a line and closes none") was written by fixer 3 (F2: "D1 drops the one-write claim and states that a large line is flushed in chunks and can interleave, as today's two-write chain can") in answer to round 3's MEDIUM on the same point. The original one-write claim was there from the start, but round 4 targets the fixer's replacement wording. | CONTINUE | Yes: fixer 4 kept the payload on today's `cat` command so the Bound holds; round 5 has no D1 write-bound finding (the bouncer judged the recurring key not a defect). |
| discussion | 4 | webster-replan-state | D8 git forms accept quoted or escaped options | B | The first D8 (commit with D8 at line 90) already admitted `git log`/`ls-remote` with arbitrary words, so the quoting bypass was there from the start. Rounds 1-3 flagged write-capable flags each time (key `d8-reader-write-flags-unbounded`), and each fixer narrowed the flag list without closing the quote/escape path, so the bound stayed false. This is an incomplete fix of a real recurring finding, not a regression. | CIRCLING | Fixed by fixer 4 (all words plain, new rejection rows); the round-4 circling decision was "accept, cause: budget"; there is no round-5 review to confirm it. |

## Plan

| perch | round | slug | finding title | class | justification | bouncer verdict | resolved next round? |
|---|---|---|---|---|---|---|---|
| plan | 4 | config-layers-run-settings | Card 15 leaves the tests that expect the old clone output unplanned | B | "In a hub nothing is ever added, and an absent one stays absent" is in card 15's first draft (commit 27b22d543, line 44), so the clone's empty prime and the stale tests existed before round 1. Rounds 1-3 reviewed card 15's clone path (R2 F1 seeding, R3 MEDIUM clone outcome) without listing the tests that read the prime's per-worktree files. | CONTINUE (circling decision: continue, cause budget) | Partly: the four named tests were added to card 15, but round 5 found three more tests under the same key (CIRCLING). |
| plan | 5 | config-layers-run-settings | Clone tests that read the prime's or a pair's per-worktree files are not carried | B | It is the same enumeration gap from card 15's first draft: `TestRunCLI_CloneEndToEnd`, `AnchoredHubScenario` and `PairInheritsConfigs` were affected from the start, and the round-4 fix enumerated only part of the set. It was an incomplete fix, not a new defect brought in by one. | CIRCLING (circling decision: accept, cause budget, settled) | No round 6; the round-5 fixer added the three tests and reran the search, but no later review checked the fix. |
| plan | 4 | logging-cleanup | Card 9's decision-read guard rows are unreachable, and one guarded site never repeats | B | The guard targets and rows for `handedLockStep`, `parentMovedUnderGate` and `undecided` are in the first draft of the card (then 08-battenshed-read-guards, commit 64956eb89), and no fixer report from rounds 1-3 touched them. Round 1 even listed card 8 as one of the cards that "reach all three branches", which misses the hard end in `awaitingStep`. | CONTINUE (circling decision: accept, cause hard-cap) | The round-4 fixer fixed it (handedLockStep moved to keep, pause-covered rows), but no review checked the fix: the run stopped at the hard cap. |
| plan | 4 | webster-dead-ends | Archive-only reset with pending findings has no way forward that makes progress | A | The round-3 fixer (F4) gave `headBeforeEveryStart` and HEAD-not-at-start the reset caller step `lyx webster accept-audit`, which is exactly the step round 4 shows cannot succeed; that F4 fix applied the round-3 reviewer's own suggested fix, built on round-2 F3, which put the pending-findings guard on the reset. The bouncer also notes it "originates in the round-3 fix". | CONTINUE (circling decision: accept, cause budget, settled) | The round-4 fixer fixed it (new decision `reset-guard-judges-against-head`), but no round-5 review checked the fix (only a focus file). |

## Webster (bolt-and-store, bugfix-fabric, bugfix-sessions, gate-test-scope)

| perch | round | slug | finding title | class | real-bug or test/doc | justification | bouncer verdict | resolved next round? |
|---|---|---|---|---|---|---|---|---|
| webster | 4 | bolt-and-store | F1 PATTERN.md exceeds the overview size cap (16430 > 16384) | B (latent, merge-induced) | doc-size cap / failing pattern test | The card 12 hub-store-gc line growth was in the original implementation, and no fixer in rounds 1-3 touched PATTERN.md. The file only went over the cap after the Publish merge-in of main (6f3760049), so earlier reviews could not have seen it fail. | CONTINUE | Yes: r4 fixer a0251d2d6 brought it to 16382 bytes, and r5 confirmed it (CONVERGED). |
| webster | 4 | bugfix-fabric | F1 switchOrForkWeft doc line 137 joins two independent clauses | C (origin A, r2) | comment style | The r2 review asked for checkout.go:137-138 to be joined onto one line, and fixer r2 did that, which created the comma-plus-so line that r4 flags. Only PATTERN-comment-line-breaks makes it gating; no code defect was found in any round. | CIRCLING | No r5 review: the r4 fixer applied the fix (fc71fd002), and the circling decision was "accept", settled. |
| webster | 4 | bugfix-fabric | F2 remoteleftover.go header line 2 and leftoverKind line 20 hold two clauses | C (origin A, r3) | comment style | The r3 review flagged remoteleftover.go:2-3 as a break inside a compound predicate, and fixer r3 put both predicates on line 2, which produced the "so Add adopts it" join (the bouncer notes this). Line 20 sits in a file that fixer r2's F3 pass also edited. | CIRCLING | Fix applied (841df36cd), but there is no r5 review; the run was accepted at circling. |
| webster | 4 | bugfix-fabric | F3 Deps.ConfigChanges/Notify docs break a compound predicate and keep a clause joined | C (origin A, r2) | comment style | The r2 review cited deps.go:126 as a "same shape recurs" site, and fixer r2's F3 break "after the comma before the conjunction" split the compound predicate across 126-127. The line 138 join was probably original text (B). | CIRCLING | Fix applied (c63604a6d), but there is no r5 review; the run was accepted at circling. |
| webster | 4 | bugfix-sessions | F1 Comment lines that end mid-phrase (4 sites) | C (origin mixed A/B) | comment style (one site is a test comment) | entryobservation.go:62 sits in the sentence fixer r3 "rewrote whole", so that site is A. seed.go:136 and smoke_test.go:640 are one-phrase substitutions in legacy paragraphs that the original diff changed and no fixer touched, so they are B. The finding is purely about line-break style. | CIRCLING | Fix applied (1d6e032a7), but there is no r5 review; the run was accepted at circling. |
| webster | 4 | bugfix-sessions | F2 Two independent clauses on one comment line (5 sites) | C (origin mostly A, r2/r3) | comment style | Fixer r2 rewrote the claudeengine.go header ("methods are pure functions"), giving the :3 site. Fixer r3 rewrote the integration_driverbootstrap_test.go sentence and broke orchengine/doc.go, giving the :22 and doc.go:231/233 sites. The orchcli/doc.go parenthetical is unclear. | CIRCLING | Fix applied (0887623b7), but there is no r5 review; the run was accepted at circling. |
| webster | 4 | bugfix-sessions | F3 Serial untagged tests edited by more than one line lack t.Parallel | C (severity inflation; origin B) | test-only hygiene | The ten tests were edited by the original diff, and no fixer touched them (fixers r2/r3 parallelized other lists). The same defect class was rated LOW in r3 (F5, watcher_test.go:910), and r4 raises it to BLOCKING by widening the rule to "edited by more than one line". | CIRCLING | Fix applied (ca4670bbe), but there is no r5 review; the run was accepted at circling. |
| webster | 4 | gate-test-scope | F1 Whole-plan validate verbs run the plan gate's type load outside a gate slot | B | real behavior bug (unslotted build, which defeats the task's slot cap) | planglyph.Validate's call to defaultTypesLoader came from the original card 9 implementation, and fixers r1-r3 never touched planglyph or the validate verbs. Rounds 1-3 missed it while they focused on test-parallel, refusal-row and gofmt issues. | CONTINUE | Yes: fixed in 97013be91 and CONVERGED in r5, although r5 LOW F3 notes that no test pins the wiring. |

## Webster (loom-bouncer-rounds, reed-multi-window, run-recovery, shuttle-turns)


Warp per-round commits are not reachable (the code repo holds only squash merges; head_sha values in the weft reports are absent from ../loomyard), so origin is judged from the fixer reports and review texts of rounds 1-3.

| perch | round | slug | finding | class | real bug or test/doc | justification | bouncer verdict | resolved next round? |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| webster | 4 | loom-bouncer-rounds | F1 malformed escalation record refuses without a way forward (circling.go:85) | B | operator-message contract (refusal text + spec row), not a behavior bug | The verb-side `refuse("%s", err)` came from batch 08; fixers 1-3 touched bouncer.go, config.go and decision.go only, and round 1 fixed the Bouncer-side twin without sweeping this sibling route. | CIRCLING (parent decided continue) | Yes: fixer 4 added ErrEscalationMalformed; round 5 confirms, though round 5 F1 (MEDIUM) finds further circling refusals forwarding helper errors. |
| webster | 4 | loom-bouncer-rounds | F2 decision add refusals with no retry clause (decision.go:82, :102) | B | operator-message contract, not a behavior bug | The status-read and generic append refusals are batch-13 code; fixer 3 only added the ErrNoDecisionsHeading mapping beside them, and round 3 reviewed decision.go without flagging them. | CIRCLING (parent decided continue) | Partly: way-forward clause added and asserted; round 5 F2 (MEDIUM) notes the two refusals still lack loom spec rows. Round 5 APPROVED. |
| webster | 4 | reed-multi-window | F1 unseen-clear set-hook targeted at bare `=session`, hook never installed (windowsize.go:186) | B | real behavior bug | Fixers 1-2 touched spawn.go, ownwindow.go and test-budget.yaml only; the bug surfaced through the real-tmux publish failure because earlier rounds ran only the fake test, which asserted the same wrong argv. | CONTINUE | Yes: round 5 has no findings, reviewer reran real-tmux tests green; CONVERGED. |
| webster | 4 | reed-multi-window | F2 OpenWindow chained options land on a same-named own window (window.go:68) | B | real behavior bug | The chain-by-exact-name in OpenWindow was original code made unsafe by the branch's own findWindowLocked change; no earlier fixer touched window.go, and it only showed up in the publish-time real-tmux run. | CONTINUE | Yes: fixer 4 used a temporary unique name plus rename; round 5 CONVERGED. |
| webster | 4 | reed-multi-window | F3 pane-gone assertion via display-message cannot detect a missing pane (ownwindow_integration_test.go:563) | B | test-only | The InvalidConfigStillRemovesAndReportsTheKey subtest was original; fixer 1 edited a different subtest (RelaunchOfADeadGroupMemberRejoinsItsWindow), and the false assertion only failed under real tmux at publish. | CONTINUE | Yes: switched to list-panes -s; round 5 CONVERGED. |
| webster | 4 | run-recovery | F1 Go comments join independent clauses at ", so" (battenshed/doc.go:20 and ~30 lines) | C (origin B) | doc/comment style | A pure comment line-break style rule rated BLOCKING; the `, so` lines existed from the start and fixer 2 explicitly left "`so`/`since` clauses as they were", so round 4 re-raises a clause form earlier rounds knowingly skipped. | CIRCLING (parent accepted) | No round-5 review; fixer 4 broke the listed lines (91c021a90) and the parent accepted at circling. |
| webster | 4 | run-recovery | F2 markdown lines join independent clauses (overview.md:477 etc.) | C (origin mostly A) | doc style | Semantic-line-break style issue rated BLOCKING; fixer 3 (1340c2e5d) rebroke the very lines in overview.md (dead-driver and adoption bullets), PATTERN-completion-signal.md, loom-status-spec.md and orch-template-role.md, leaving the `, so` joins in place. | CIRCLING (parent accepted) | No round-5 review; fixer 4 broke the listed lines and the parent accepted. |
| webster | 4 | shuttle-turns | F1 offset-seeding sentence breaks mid-phrase (attach.go:281) | C (origin B) | comment style | Comment-wrap style rated BLOCKING; the sentence's tail was rewritten by the original diff, and fixer 1 reflowed attach.go but treated this legacy-wrapped sentence as exempt. | CIRCLING (parent accepted) | No round-5 review; fixer 4 reflowed it (22f06b1f4), parent accepted. |
| webster | 4 | shuttle-turns | F2 TestAttach_OffsetStartsAtZero doc breaks mid-phrase (attach_test.go:904) | C (origin B) | test comment style | Comment-wrap style in a test doc comment; the tail was rewritten by the original diff and no earlier round listed it. | CIRCLING (parent accepted) | No round-5 review; fixer 4 fixed (8c8e14d00), parent accepted. |
| webster | 4 | shuttle-turns | F3 TestEngine_RetrySemantics doc breaks mid-phrase (treadleengine/engine_test.go:261) | C (origin B) | test comment style | Comment-wrap style; fixer 2 explicitly left the treadle test headers as word-swap legacy wraps, and round 4 reclassified this one as a rewritten clause. | CIRCLING (parent accepted) | No round-5 review; fixer 4 fixed (17b7f87fa), parent accepted. |
| webster | 4 | shuttle-turns | F4 smoke_guardrail_test.go header breaks mid-phrase (:3) | C (origin B, re-raise) | test comment style | Fixer 1 explicitly exempted this header as legacy text with substituted words, and fixer 4 notes round 4 simply re-read the same tail as a rewrite; a re-judgment of an earlier exemption, not a new defect. | CIRCLING (parent accepted) | No round-5 review; fixer 4 reflowed (522a2d1fe), parent accepted. |
| webster | 4 | shuttle-turns | F5 settings.go header breaks mid-phrase (claudeengine/settings.go:1) | C (origin A) | comment style | Fixer 3 (9d0a4fe1c) moved this header's rewritten final clause onto the preceding line to remove a fragment, and round 4 flags the same sentence for still breaking mid-phrase; style issue rated BLOCKING. | CIRCLING (parent accepted) | No round-5 review; fixer 4 fixed (c8157ced4), parent accepted. |

Pattern: the only real behavior bugs (reed-multi-window F1, F2) and the one false test (F3) were present from the start and surfaced only when real tmux ran at publish, so earlier fake-only reviews could not see them.
The run-recovery and shuttle-turns late blockers are all comment/markdown line-break style rated BLOCKING; fixers cleared listed sites one at a time, deliberately exempting some forms, so the next reviewer re-raised the leftovers and the loop CIRCLED; loom-bouncer-rounds is the same enumeration gap for refusal way-forward text.

</details>

