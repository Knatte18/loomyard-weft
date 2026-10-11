# Support log: per-role-models

## Interview

The session ran autonomously: no operator answered, and every decision below is a self-pick recorded in the question ledger.

1. Read the board entry `per-role-models` and the entries that touch it: `haiku-roles` (judge stays on Sonnet, 2026-10-11; seed rated Haiku-safe), `weaver` (fork-based Webster stays; cold-start implementer is separate), `fan-master-preread` (lens forks run on the fanned reviewer's model), `config-overlay` (unbuilt repo-wide config layer; no conflict), `burler-multi-llm` (Burler round rewiring; untouched here).
2. Explored the code at module level: the run-wide `judge` key flows from `loomengine` through the loom wiring onto the recipe environment, and the Bouncer entry falls back to it when its recipe row sets no `model`; both the seed and the judge spawn read the same Bouncer model fields.
   No embedded recipe's Bouncer row sets `model`, `effort` or `version`.
3. PR-Rework parses the `plan` key and reuses `plan_timeout_min`.
4. Re-checked the role survey against every config-driven model-spec parse site: webster's `master`/`recovery`, darn's `writer`, landing's `conflict`/`describe`, loom's `discussion`, `plan`, `driver`, `friction`, seats advisors and the review lists each already have a key; the only shared roles in scope are the judge, the seed and PR-Rework.
5. Read the judge's decision rule and ledger contract: any BLOCKING finding in the latest review rules CONTINUE, a BLOCKING finding is never relabelled downward, and the ledger must carry every previous key forward; Go parses the ledger but compares nothing against the previous one.
6. Found the retired-verdict route (archive the three judge outputs, degrade Stuck, re-judge, one bounce unit) as the precedent for refusing a judgment without a new loop.
7. Read the usage record: written once per completed Burler round, read for fan and lenses by the facts and focus code; no judge or seed entry exists, and the token report under `tools/tokencount` reads transcripts by session role.
8. Settled D1–D11 in the decision record.

## Rejected alternatives

- A `seed` template default of `haiku[medium]`: it would move every worktree's seed passes to Haiku on landing; the parent ruled the task adds keys and changes no default model, and the Bouncer's Haiku work is dropped.
- One run-wide `seed` key with no per-segment layer: cheaper by three template lines, but asymmetric with the judge keys and short of the board's "each Bouncer row judge and seed pass".
- Per-segment keys replacing the run-wide `judge` outright (no fallback): breaks every existing `loom.yaml` and departs from the review/fix convention.
- Keeping the recipe-level `model`/`effort`/`version` on the Bouncer row, with config as fallback: one recipe key cannot name both passes, nothing sets it, and a recipe literal outranking config is the problem the board names.
- Per-round lists for the judge and seed keys, as the review and fix keys have: no stated need.
- A `rework_timeout_min` key: the board asks for model keys only.
- Overriding a refused CONVERGED to CONTINUE in Go: the next round would get the CONVERGED pass's empty focus file and a ledger Go has just distrusted, and Go would become a second writer of the judge's files.
- Halting the run on a refused judgment: a single careless judge pass would stop a run that a re-judge can recover; the bounce budget already bounds repeats.
  The one exception is a round whose budget is already reached, where the refusal's counted Stuck meets Shed's budget-exhausted block at once (D9).
- Escalating a refusal on a round whose budget is reached, as a CONTINUE there escalates: the refusal has archived the round's judgment, so the escalation would have no ledger to point at and no judged round to settle the parent's decision over without new state; Shed's block and its `goto` way forward recover the run.
- Feeding the refusal reason into the re-judge prompt: extra state and stencil surface for a backstop that should rarely fire.
- Checking the ledger-side BLOCKING only, per the board's wording: a judge that omits a BLOCKING finding from its ledger would still pass; the review-side check closes that, and matches the judge's own decision rule.
- Checking the fixer report's disputes in the guard: the judge reads no fixer report, so the guard would hold the judge to evidence it never saw.
- Separate usage files per judge and seed pass (`round-<N>-judge-usage.yaml`): avoids the merge-on-write, but splits a round's cost across files, against the board's "in the usage files like other roles".
- One judge entry per round, overwritten on re-judge: undercounts a re-judged round's cost.
- Changing `tools/tokencount` to read usage records: it already counts judge and seed sessions by role from transcripts; out of scope.

## Review rounds

_No rounds yet._

### Parent review, round 1 (BLOCKING)

- F1 [BLOCKING, scope] D3 set the `seed` template default to `haiku[medium]`, moving every worktree's seed to Haiku on landing — fixed: D3 and A2 dropped; D2 now gives `seed` and the per-segment seed keys an empty default that falls back to the segment's resolved judge model; the Scope "Out" exception and the Open risks bullet removed; the acceptance criteria state that with every new key empty a seed runs on its segment's judge model, and the `plan_judge` criterion now reflects the fallback.

## Question ledger

- Q1 (resolved, auto-pick): per-segment keys for both the judge and the seed, layered over run-wide `judge` and `seed`? Yes — mirrors `*_review`/`*_fix` over `review`/`fix`; D1, D2.
- Q2 (resolved, parent ruling in parent-review round 1): seed default `haiku[medium]` now, or keep today's model? Keep today's model: `seed` and the per-segment seed keys default empty and fall back to the segment's resolved judge model; the board's Haiku line for the seed is stale and the task adds keys only; D2. The earlier auto-pick of `haiku[medium]` (former D3) is withdrawn.
- Q3 (resolved, auto-pick): retire the Bouncer row's recipe `model`/`effort`/`version` keys? Yes; D5.
- Q4 (resolved, auto-pick): `rework` empty falls back to `plan`, and no timeout key? Yes; D6.
- Q5 (resolved, auto-pick): what does Go do with a CONVERGED over an open BLOCKING finding — override, halt, or re-judge? Re-judge via the retired-verdict route, one bounce unit per refusal; D7, D9.
- Q6 (resolved, auto-pick): source of "open BLOCKING" — ledger, review, or both? Both, review parse failure included; D7.
- Q7 (resolved, auto-pick): does the lost-key check apply to every verdict or CONVERGED only? Every verdict from round 2, skipped when the previous ledger is malformed; D8.
- Q8 (resolved, auto-pick): usage shape — same file with lists, or separate files? Same `round-<N>-usage.yaml`, `seed` and `judge` lists, each writer replacing only its part; D10, D11.
- Q9 (resolved, auto-pick): does `tools/tokencount` change? No; it reads transcripts by role.
- Q10 (resolved, auto-pick, review round 2): at a reached bounce budget, does a refusal escalate to the parent or meet Shed's budget-exhausted block? Shed's block, with no re-judge; D9.
- No tool failed and nothing outside the two output files looked broken.
