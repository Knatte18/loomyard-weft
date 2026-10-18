# Audit: Discussion and Plan quality before and after the twill writers

`twill-writer-quality-audit` · Note · audit, loom, burler, stencil

## Question (operator, 2026-10-18)

Has the quality of Discussion and Plan output gone up since the writers became twill-based?
An agent answered it on 2026-10-18 from the records (weft archive tags, Bolt config history, `lyx report convergence`); below is its finding.

## Short answer

Plan output got better when its writer became twill-based.
Discussion is the same or slightly better, which the data cannot show with confidence.
Adding the fork0 plan writer and the code list (generation 3) shows no further gain.

## Method

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

## Signals

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

## Verdicts

- Plan, G1 to G2: better, moderately high confidence. Round-1 BLOCKING+MEDIUM fell about 50% (about 3 standard errors even against the later G1 runs), in every size bucket, concentrated in the defect kinds the twill-plan lenses check; part of the format drop is glyph format settling.
- Plan, G2 to G3: same, low to moderate confidence. The bench predicted sharper fork0 lens catches (`twill-fork0-bench.md`); production shows no extra drop. The review rubric also changed in G3 (`9ea1f7474`, `e131c7625`, `27657341c`), and the code list (`530c339e9`) is mainly a cost change.
- Discussion: same or slightly better, low confidence. The 0.86-finding drop is about 2 standard errors, rounds did not move, the finding kinds did not change, and G3 is smaller tasks with the same findings per card; this matches `twill-discussion-bench.md` (contaminated, inconclusive) and `twill-discussion-fork0-bench.md` (keep twill, no fork0).

## Data that would settle the open parts

1. Blind real/noise grading of round-1 findings, as the bench grading did, on about 10 matched runs per generation.
2. A production replay: single writer against twill on 5 to 10 landed tasks from the solo-review window, with the same reviewer.
3. Plan-Write transcripts, to count `validate-plan` failures inside the session.
4. More runs under the fan review, for a post-`112cccb8f` baseline.
5. Diff sizes: for 113 of 179 runs the batch commits cannot be resolved in the prime, so only cards normalize size.
