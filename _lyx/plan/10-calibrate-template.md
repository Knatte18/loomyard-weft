# Card 10 — set the cautious profile from the calibration report

**Edit:**
- `internal/batcher/template.yaml`

**Uses:**
- `tools/tokencount/calibrate.go`

**Intent:**
Run the calibration report once on the operator's transcripts and, when the fit is good, rescale the `cautious` profile's context-size numbers by one factor (Auto-mode assumptions).
`active` stays on identity.

Run, from the worktree root:

```
go run ./tools/tokencount -last 40 -calibrate cautious -history ../card-batching-weft
```

`-history` names the sibling checkout that holds the hub's `loom: plan artifacts for <slug>` commits and archive tags; `-config` defaults to the worktree root, whose `batcher.yaml` resolves to the template when absent.

Read the overall fit summary.
The fit is good when at least 10 cards were measured and the spread (75th percentile of the ratios over the 25th) is at most 3.

- Good fit: the scale factor `s` is the overall median ratio of measured weight to estimate.
  Multiply `startup_context`, `context_per_line` and `package_context`, the only coefficients that carry a context size, and `alone_above` and `budget` by `s`, each rounded to three significant digits; `fork_messages` and the per-target, per-test-file and per-`Uses` message coefficients stay as they are, since one-card forks cannot identify the per-fork messages.
  Every estimated cost then scales by `s`, and the thresholds stay in the same unit as the costs.
  The commit body records the run count, the measured card count, the median and the spread.
- Poor fit, or a report that cannot be produced: change nothing, make no commit, and state the reason and the fit figures in the batch report.

**ImpactSummary:** only the inactive `cautious` profile's numbers can change; `active` stays on identity, so no run's batching changes.
