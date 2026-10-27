# Audit the twill and lens review setup: worth per perch, recurring finding classes, writer quality

`review-setup-audit` · Note · research, burler, loom, report, stencil, shed

## Questions

One audit over the same records (review files in the weft archive tags, `lym report convergence`, `tools/tokencount/tokencount.sh`) answers three questions:

1. Is the lens fan and twill fixer setup worth its cost at each perch, and what should the standard review rounds be?
2. Which finding classes recur at Plan and Discussion, and where is each cheapest to stop at the source?
3. The open parts of the twill writer quality audit below, under "Data that would settle the open parts".

## When (operator, 2026-10-22)

Run it once about ten loom runs with the current setup have landed on Neve; runs from OSL-1046 are not gathered for it.

Later the same day (operator): run it now, over the loom runs landed since `504c5552` (2026-10-20), with the runs after `research-in-discussion` and `lenses-language-sets` (2026-10-22), which change Discussion, reported apart.

## Result of the first run (2026-10-22)

A sonnet agent ran the audit over the 6 loom runs landed since `504c5552`; the sample is small and only `finalize-dirty-names-paths` ran Discussion after the 2026-10-22 changes.

- Late rounds still pay: discussion round 3 found something real in 3 of 3 runs that reached it, plan round 2 in 3 of 3, nearly all new findings rather than too-narrow earlier fixes, so lowering caps is not supported.
- The lens fan raised 93% of round-1 gating findings at Discussion and 58% at Plan, for about $1 a perch; a review perch costs about $10-11, mostly in its later rounds.
- No further cut in round-1 findings from twill-plan is visible, and none is ruled out.
- Agent recommendation, not a decision: template defaults Discussion 2 soft / 3 hard, Plan 1 soft / 2 hard, Webster 1, Merge 1 (profile `light`'s values).
  Since `fed963ec5` these are no longer `light`'s values: the ladder is `speed`, `light` (1 round per perch), `solid` (Discussion 2, Plan 2) and `wide`.

## Operator decision (2026-10-22): lens fan in Discussion round 2

Since late rounds still find new things, Discussion-Review round 2 also runs the lens fan, and round 3 on runs solo: the hub's `loom.yaml` sets `discussion_reviewer_fan: ["review-discussion", "review-discussion", ""]`.
The twill fixer stays in every round, since many late-round defects are introduced by a sloppy fixer.
The next run of this audit checks what round 2's fan finds against a solo round 2, before Plan gets the same.

## Operator view, not a decision (2026-10-22): fewer, sharper lenses

Tokens could be saved by scoping each lens more precisely and having fewer of them.
A later run of this audit, with more material, measures per lens what it finds that no other lens or the plain reviewer finds, and how much of it is real, so lenses can be merged, sharpened or dropped.
The Plan writer's `twill-plan` fan comes first: the Plan lead carries a large context, and each of its lens forks inherits it, while the Discussion lead's context stays small.

## Review rounds after twill-plan (was note `review-rounds-after-twill-plan`)

The earlier audit below found discussion-review round 3 and plan-review round 2 useful only when an earlier round's fix was too narrow.
Since `504c5552` (2026-10-20), the Plan writer's `twill-plan` fan runs every `review-plan` lens, as the Discussion writer's fan already runs every `review-discussion` lens, which should cut round-1 findings further.
Compare those runs' round-1 Plan findings against the earlier audit's baseline, and check whether discussion-review round 3 and plan-review round 2 still find anything; if they rarely do, lower the default review bounce caps for those segments.

## Review setup

### Seen (2026-10-18)

- On 2026-10-16/17 every perch got the reworked review setup: a lens fan in round 1 (`review-discussion`, `review-plan`, `review-code` beside a plain reviewer), a twill fixer every round, and twill writers for Discussion and Plan.
- The setup costs tokens and time at every perch, and no measurement yet shows what it catches that the plain setup missed.

### Operator decision (2026-10-18)

- Audit the use of the twill and lens setup across every perch once it has run for a while, and set the standard review rounds from the result (see `run-profiles`).

### Orch recommendation, not a decision

- Run it after about ten loom runs with the setup, from `lym report convergence` (stop classes, fresh-code fixes, review rounds per segment), `tools/tokencount/tokencount.sh` and the review files in the records repository.
- Per perch: how many findings each fan member and round raised, how many led to a change that stayed, how many the later perches or PR-Gate would have caught anyway, and the tokens and minutes each round cost.
- Compare with the runs from before 2026-10-16 where the records allow it.

## Plan writer finding classes (was note `plan-writer-finding-classes`)

### Seen (2026-10-15)

Over the 15 most recent full loom runs (`tools/tokencount/tokencount.sh -last 15`), Plan cost about $13 a run: the plan lead about $5.50, its forks (fork0 and the five lens forks) about $4, and Plan-Burler-Review about $3.80.
Each fix the lenses or the review ask for is work the writer did not get right the first time.

### Operator view, not a decision (2026-10-15)

Plan is heavy by nature, since it turns the Discussion into concrete work; the aim is a smarter Plan writer, so fewer Burler rounds are needed.

### Idea (orch recommendation)

- A fresh agent classifies every finding the Plan lenses and Plan-Burler-Review raised in recent runs: what kind of defect, whether it was real, and whether the review found something the lenses had not.
- Each recurring class goes where it is cheapest to stop at the source:
  - a machine check in `lym loom validate-plan`, when the class can be checked deterministically;
  - a writer rule in the Plan stencil, when it is a judgement the writer can make up front;
  - a missing input, when the writer lacked something it needed (for example what `discussion-relevant-list` now hands it).
- Then measure whether lens and review findings per run drop.

### Operator view, not a decision (2026-10-16)

The lens fan and twill fixer (`fan-prompt-tuning`) are built first.
Later, this audit runs over real runs made with them, and the wording of the reviewer and fixer prompts is adjusted from what it observes, beside the Plan writer measures above.

### Operator decision (2026-10-16)

The Discussion and Plan perches run the twill fixer in every round; the lens fan reviewer runs in round 1 only, with solo review from round 2.
Reason: fixers introduce many defects, and a defect a late-round fixer introduces is seen only by the next review, or not at all at the hard cap.
The hub's `loom.yaml` carries it as `discussion_fan: ["review-discussion", ""]`, `plan_fan: ["review-plan", ""]`, `discussion_fix_fan: fix-discussion` and `plan_fix_fan: fix-plan`.
Orch recommendation, not a decision: the audit compares late-round fixer defects against runs from before this setup.

### Moved from note `perch-review-analysis` (issue #676, 2026-10-16)

The analysis measured rounds, tokens and late BLOCKING findings per perch; its classification of the late BLOCKING findings moved to `fan-prompt-tuning`, the lens fan and twill fixer it proposed are built there, and its proposal of two parallel round-1 reviewers is replaced by the lens fan.
Its Webster proposal (a hard cap of 2) is already met by the hub's `loom.yaml`: `webster_max_bounces: 1` with `webster_auto_accept: true`.
Two of its proposals are left for this audit to weigh, numbered as in the original:

3. **Later rounds review the fix delta.**
   A findings cannot be caught in round 1; they need a round after the fix.
   From round 2, one reviewer checks only what the fixer changed plus the sibling decisions or cards it touches, which is cheaper than a full review.
   Compare Traycer's incremental re-verify in #671.
6. **Measure again in a week**, with the same scripts: rounds per perch, BLOCKING share in round 1, and tokens per perch, before and after twill and the changes above.

## Twill writer quality, earlier result (was note `twill-writer-quality-audit`)

### Question (operator, 2026-10-18)

Has the quality of Discussion and Plan output gone up since the writers became twill-based?
An agent answered it on 2026-10-18 from the records (weft archive tags, Bolt config history, `lym report convergence`); below is its finding.

### Short answer

Plan output got better when its writer became twill-based.
Discussion is the same or slightly better, which the data cannot show with confidence.
Adding the fork0 plan writer and the code list (generation 3) shows no further gain.

### Method

The main signal is round-1 BLOCKING plus MEDIUM findings of landed loom runs, read from the front matter of each run's `round-1-review.md` in the weft archive tags.
Each run is put in a generation by when its writer row started, checked against prod deploys and the hub config history on Bolt:

| Change | Live from | Set by |
|---|---|---|
| twill Discussion writer | 10-11 15:51 | `afd52a6f6` |
| twill Plan writer | 10-12 10:25 | `6099cd2bf` |
| fork0 plan writer (`2abb06c5e`) | 10-15 08:35 | deploy |
| code list (`530c339e9`) | 10-15 18:13 | deploy |
| review fan (`112cccb8f`) | 10-16 16:40 | `e57361ba5` |

Confounders and how they were handled:

- Baseline window 10-08 21:38 to 10-16 16:40, where every round-1 review ran solo on `opus[medium]` with no fan; earlier runs flipped reviewer between sonnet and opus (BLOCKING rate 0.69 against 1.32 per run) and are left out, as is `webster-replan-state`, which had a fan.
- After `112cccb8f` only five runs exist (`pr-gate-run-identity`, `creel-cli-output-fixes`, `shedrecipe-capabilities`, `creel-notices`, `orch-rules-files`); they are reported apart, not compared. The fan raises counts about 1.6x (5.4 to 8.8 findings per run at Discussion), matching 14 against 9.5 in `burler-fan-tuning-bench-2026-10-16`.
- `a273d69bc` only renamed keys; the models stayed the same (`25cc04048`).

### Signals

G1 is before twill, G2 twill, G3 fork0 plus code list.

Round-1 BLOCKING+MEDIUM per run (± standard error), and runs with any BLOCKING:

| Segment | G1 | G2 | G3 |
|---|---|---|---|
| Plan | 3.77±0.40, 26/40 runs | 1.72±0.30, 9/29 | 2.03±0.28, 12/33 |
| Discussion | 4.43±0.27, 20/28 | 3.57±0.27, 39/58 | 2.93±0.30, 6/15 |

- The first Plan runs (10-10 morning) are inflated by the switch to glyph-based plans (`78a2b1f14`); with Plan runs from 10-10 13:00 only, G1 is still 3.17.
- By task size in cards (diff size could not be resolved): the Plan drop holds in every bucket, 5.6 to 3.0 (over 12 cards), 2.9 to 1.7 (7 to 12), 2.2 to 0.4 (6 or fewer); G3 sits at or slightly above G2 (3.8, 2.5, 1.3). Discussion drops less: 5.3 to 4.8, 4.2 to 3.6, 3.8 to 2.3. G3 tasks are smaller (6.6 to 7.3 cards against about 10), and per card G3 Discussion is back at baseline (0.44 against 0.46).

Plan finding kinds per run, keyword counts over summaries, checked by reading about 15 runs:

| Plan finding kind, per run | G1 | G2 | G3 |
|---|---|---|---|
| design class | 1.43 | 0.62 | 0.48 |
| duplicate or over-specified tests | 0.38 | 0.03 | 0.15 |
| edits or tests no card targets | 0.35 | 0.14 | 0.12 |
| gate or glyph format | 0.75 | 0.31 | 0.12 |
| test branch no planned test reaches | 0.28 | 0.17 | 0.18 |

G1 runs such as `quick-recipe`, `config-layers-run-settings` and `driver-arbiter` are full of duplicate-test and missing-refusal-row findings, which the twill-plan lens forks target.
Discussion findings are the same kind in every generation, design holes such as unstated bounds and false premises (`lyx-halt`, `convergence-plan`, `fan-prompt-tuning`); design-class findings fall from 3.4 to 2.3 to 1.8 per run.
Wrong code references are rare in every generation and were not separated.

Rounds to converge, and circling settled by decision:

- Plan: 2.10 (5/40 circling), 1.69 (4/29), 1.61 (3/33).
- Discussion: 2.43 (10/28), 2.50 (21/58), 2.27 (2/15).
- Only `shuttle-send` (before 10-08) hit a hard cap.
- Rounds are confounded by Bouncer changes (`0ebdea27f` 10-08, `b2e4cf913` 10-11, `072e2ed86` 10-13, `85a8f3434` 10-14) and by sonnet reviewers in later rounds from 10-14 20:16 (Plan) and 10-15 07:30 (Discussion); within G2 circling halves after `072e2ed86` (14 to 7) with no writer change.

Plan gate failures: the Go gate retried Plan-Write once in the whole set (`tender`); the twill writers run `validate-plan` inside their session, so their failures are not in the records; no Plan-Write reran after 10-10 except `discussion-multi-llm`.

Downstream:

- Webster stops: G1 5/40 (`daemon-wakeups`, `test-suite-weight`, `driver-context`, `cli-surface`, `quick-recipe`), G2 2/29 (`tender`, `overview-seats` with plan_drifted), G3 0/33.
- Webster review rounds 1.98, 1.48, 1.12, but many Webster changes landed in the same window, so this is not credited to the writers.
- Webster deviations left out: their definition changed twice (`30385e1ae`, `493c62b71`).
- PR-Rework: G1 2/40 (`discussion-multi-llm`, `reed-hygiene`), G2 3/29 (`convergence-plan`, `reed-window-order`, `twill-fork0`), G3 1/33 (`creel-core`); no trend.
- Decisions added after Discussion: G1 11/28 runs (33 entries), G2 17/58 (37), G3 0/20; mostly parent design calls at circling escalations, which fell when `072e2ed86` and the alignment agent (`a40c7baef`) landed, so no writer signal.

### Verdicts

- Plan, G1 to G2: better, moderately high confidence. Round-1 BLOCKING+MEDIUM fell about 50% (about 3 standard errors even against the later G1 runs), in every size bucket, concentrated in the defect kinds the twill-plan lenses check; part of the format drop is glyph format settling.
- Plan, G2 to G3: same, low to moderate confidence. The bench predicted sharper fork0 lens catches (`twill-fork0-bench.md`); production shows no extra drop. The review rubric also changed in G3 (`9ea1f7474`, `e131c7625`, `27657341c`), and the code list (`530c339e9`) is mainly a cost change.
- Discussion: same or slightly better, low confidence. The 0.86-finding drop is about 2 standard errors, rounds did not move, the finding kinds did not change, and G3 is smaller tasks with the same findings per card; this matches `twill-discussion-bench.md` (contaminated, inconclusive) and `twill-discussion-fork0-bench.md` (keep twill, no fork0).

### Data that would settle the open parts

1. Blind real/noise grading of round-1 findings, as the bench grading did, on about 10 matched runs per generation.
2. A production replay: single writer against twill on 5 to 10 landed tasks from the solo-review window, with the same reviewer.
3. Plan-Write transcripts, to count `validate-plan` failures inside the session.
4. More runs under the fan review, for a post-`112cccb8f` baseline.
5. Diff sizes: for 113 of 179 runs the batch commits cannot be resolved in the prime, so only cards normalize size.
