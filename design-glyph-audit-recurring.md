# Recurring Webster kick-start pack (glyph) audit

`glyph-audit-recurring` · Note · audit, webster, quarry

## What this is

A recurring audit of how Webster implementer forks use their kick-start pack (glyph spans) and whether the pack is correct, run with `tools/forkaudit/forkaudit.sh --since <date> --pack-check --plan`.
Each audit keeps its report and summary under `docs/research/glyph-audit-<date>/`.
Operator instruction 2026-10-17: keep this on the board so future glyph audits are not forgotten.

## Audits so far

- `docs/research/glyph-audit-2026-10-12/`, `-10-13/`, `-10-15/`: the inline, read-all-first and pack-file layouts.
- `docs/research/glyph-audit-2026-10-17/`: the first audit of the per-card layout (`lyx loom card-pack`, default since `aa925bdcc`), 90 forks from 26 runs.
  Card forks read about half their spans before the first edit (pack-file forks a quarter) and start editing after half as many calls; 0 stale spans of 1661 checked at batch start; about 20% of edits still go to files with no span; cost per card is higher, but the workloads differ.

## Open points for the next audit

- `forkaudit` grades only a fork's first `card-pack` call, so files a later card's pack covered count as missed, and the 20% missed share is an upper bound (open finding from `aa925bdcc`, in `tools/forkaudit/pack-entries.jq` and `report.jq`).
- The card arm's spans-window definition may bias the spans-reached share (open finding from `aa925bdcc`, in `tools/forkaudit/extract.jq`).
- Pack drift during a batch is unmeasured.
- Five forks in the 2026-10-17 sample matched no arm marker (`none`).
- Three card forks had no spans at all, among them a creel-core fork with 81 edits; why is unknown.

## Orch recommendation, not a decision

Run the next audit after about 20 more Webster runs, or right after any change to the pack layout or the fork prompt, whichever comes first, and fix the first-call grading gap in `forkaudit` before it, so the missed-files share means something.
A cheap model (Sonnet) runs the script; the orch places the result in `docs/research/`.
