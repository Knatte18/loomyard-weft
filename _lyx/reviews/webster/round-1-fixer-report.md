# Webster round 1 — fixer report

Both findings were validated against the code and fixed, one commit each.

## Fixed

- **F1** (LOW, `docs/research/haiku-judge.md`) — commit `bbc4e1e17`.
  Valid: the note named the retired `Env.JudgeModel`/`JudgeEffort`/`JudgeVersion` fields, a single run-wide `judge` key and per-row `model`/`effort`/`version` recipe keys.
  "How the judge's model is set" now describes `judge`, `seed` and the per-segment judge and seed keys, their fallback, `ResolveJudge` at run start, threading through `Env.RowBouncerModels`, and `bouncerEntry` refusing the row keys.
  The other statements the diff falsified were updated too: the seed line (it no longer moves only with `judge`), the claim that Go does not cross-check verdicts, which also appeared in the verdicts summary, the trial's "switch all three rows" step, which now sets `plan_judge`, `webster_judge` and `seed`, and the closing proposal of a Go check that card 5 built.
  The measurement sections were left unchanged.
- **F2** (NIT, `internal/shedadapters/bouncer.go`) — commit `0bd8d9f35`.
  Valid: `readJudgment` already requires round N's ledger to read, parse and name round N before `recordedVerdict`/`harvestedVerdict` return true, so the guard's own ledger re-read could never fail.
  `readJudgment`, `recordedVerdict` and `harvestedVerdict` now return the parsed `ledgerFile` beside the verdict.
  `settleUnlessRefused` and `judgmentRefusal` take that ledger, so the unreachable "ledger is unreadable" and "ledger does not parse" refusals are gone.
  `rejudge` takes the verdict instead of re-reading it.
  The other callers and four test call sites changed to the new return shape only.
  No test was added, since this was not a coverage gap.
  `lyx gate test ./internal/shedadapters` and `lyx gate test --tags integration ./internal/shedadapters` pass, and `lyx loom lint-comments` passes.

## Disputed

None.

## Deferred

None.
