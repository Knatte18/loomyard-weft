# Card 3 — arm-tables

**Edit:**
- `tools/forkaudit/report.jq`
- `plan:tools/forkaudit#TestForkauditReport`

**Uses:**
- `docs/research/glyph-audit-2026-10-15/stats-pf.jq`
- `docs/research/glyph-audit-2026-10-15/quartiles.jq`

**Intent:**
Add the per-arm pack-use table, the arm comparison table and the exploration quartiles to the report, with the missing-metric rule (decision D5's tables).

`report.jq`, between the samples table and the per-fork table:
- Pack-use table, one column per arm with forks: medians per fork and pooled values of the D4 pack metrics as `stats-pf.jq` computed them (pack opened, its turn, before the first edit, spans per fork and per card, strict and file-level span reach before the first edit, span reach in the first work turn), and the count of forks reaching at least half their spans (strict) before the first edit.
- Arm comparison table: medians per card and pooled per card (sum over sum of cards) of exploration calls raw and net, whole-file reads raw and net, partial reads, greps, member greps, quarry lookups, calls before the first edit net, turns, output tokens, fresh input tokens and cache reads, plus the share of forks with three or more parallel calls in the first work turn.
- Quartiles (min, q1, median, q3, max, nearest rank as `quartiles.jq`) of net exploration calls per card per arm.
- Missing-metric rule: a fork lacking a metric is left out of that metric's median and pooled value — no edit leaves out every "before the first edit" metric, a pack with no spans every span share, arm `none` every pack metric, zero cards every per-card metric.
  A cell whose sample is smaller than the arm's fork count shows its size as `<value> (n=<k>)`, and a cell with no sample left shows `n/a`.
- Numbers round to two decimals, shares to whole percent, as `stats-pf.jq` printed them.

`TestForkauditReport`'s `--since` row gains assertions on cells hand-computed from the fixtures:
- the `file` arm's "before the first edit" cells carry `(n=1)` because `fork-file-noedit` has no edit;
- the `inline` arm's span-share cells read `n/a` because its pack has no spans;
- the `none` arm's pack-use cells read `n/a`;
- the `unknown` arm's per-card cells read `n/a` because `fork-unknown` has zero cards;
- the `file` arm's count of forks reaching at least half their spans, its share of forks with three or more first-turn parallel calls, its pooled per-card exploration calls (sum over the sum of cards, which differs from the mean of per-fork ratios for the two `file` fixtures), its turns and output-token medians per card, and its exploration median and quartile row match the hand-computed values.

Its `--runs alpha` row gains assertions that `inline`, `none` and `unknown` have no column or row in the pack-use, arm comparison and quartile tables.

**ImpactSummary:** report rendering only; two existing table rows gain assertions, no new test and no change outside `tools/forkaudit`.

**Verify:**
lyx gate test --tags integration ./tools/forkaudit
