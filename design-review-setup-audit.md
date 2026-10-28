# Audit the review setup: rounds per perch, per-lens worth, finding classes

`review-setup-audit` · Task · research, burler, loom, stencil · done

## Question

One audit over the records of landed loom runs answers what each part of the review setup is worth today:

1. **Rounds per perch.** Per perch (Discussion, Plan, Code), what each round finds that stays fixed, against its tokens and minutes; in particular what Discussion-Review round 2's lens fan finds against a solo round, and whether Discussion round 3 and Plan round 2 still find new defects. The answer sets the profile ladder's defaults (`lym board profiles`).
2. **Lenses.** Per lens, for `twill-plan`, `twill-discussion`, `review-discussion`, `review-plan` and `review-code`: what it finds that no other lens or the plain reviewer finds, how much of it is real, and which lenses can be merged, sharpened or dropped.
3. **Finding classes.** Which classes recur at Discussion and Plan, and where each is cheapest to stop: a machine check in `lym loom validate-plan`, a writer rule in a stencil, or an input the writer lacked.
4. **Fix-delta review.** Whether a round from round 2 on that reviews only what the fixer changed, plus the decisions or cards it touched, would have caught what the full later rounds caught.

## Scope and cap

- Runs: the loom runs landed on Neve since `79bab47a0` (`lens-fork-reports-kept`, 2026-10-22), whose per-lens reports are kept verbatim as `*.lens-<lens>.md` under `_lym/plan/`, `_lym/discussion/` and the review rounds in the weft archive tags; runs from OSL-1046 are left out.
- Sources: those files, the rounds' review files, `lym report convergence`, `lym report timing` and `tools/tokencount/tokencount.sh`.
- No agent or model run beyond the darn writer itself: no bench, no replay, no grading agents. Where real-or-noise needs a judgement, the writer judges it and says so.
- Output: `docs/research/review-setup-audit-2026-10-28/report.md`, with the sample size beside every number and a recommendation per question, marked as a recommendation, not a decision.

## Baseline from earlier runs of this audit (2026-10-22)

- Over 6 loom runs, late rounds still paid: Discussion round 3 found something real in 3 of 3 runs that reached it, Plan round 2 in 3 of 3; the lens fan raised 93% of round-1 gating findings at Discussion and 58% at Plan, for about $1 a perch, against about $10-11 a perch.
- `twill-plan` went from 8 lenses to 5 (`coverage`, `row-validity`, `forced-edits`, `guard-reach`, `way-forward`) in task `twill-plan-five-lenses`.
- Low-confidence proposals then, to test now with per-lens data: `twill-discussion` from 7 lenses to 4 (`decision-chain`, `code-claims`, `board-scope`, `leanness`); `review-code`'s `code-affected-set` folded into `code-pattern-fit`; `review-plan` given the lenses the writer fan lacks instead of rerunning its three.
- Current hub setup: the lens fan runs in Discussion-Review rounds 1 and 2 and Plan-Review round 1, solo after; the twill fixer runs every round (`loom.yaml`'s `discussion_reviewer_fan`, `plan_fan` and the fix fans).
- The twill writer quality audit of 2026-10-18 found Plan output better with the twill writer and Discussion the same or slightly better; Bolt's history of note `review-setup-audit` holds its tables.
