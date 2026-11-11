# Report: the advisor prompt tuning stopped before any rerun

Adds `docs/research/loom/advisor-model-trial-2026-11-11/report.md` and its link in `docs/research/README.md`.
The report holds no iteration: no route sets Claude Code's advisor for one shuttle run, so none ran.

## What the report records

- The advisor's settings key: Claude Code 2.1.322 reads `advisorModel` and a `--advisor` flag.
- The earlier seats result and why it does not predict an advisor's cost.
- The recorded `reed-tab-and-attach-fixes` run the reruns would compare against.
- The blocker: the `--settings` document `claudeengine` writes has no advisor key and `lym shuttle run` has no flag for one;
  the heddle-rendered `.claude/settings.local.json`, a heddle entry and a `binary` wrapper would each reach more than one run.
- That the prompt is not ready to bench.

## Design decisions

- No rerun was made and no rendered file was edited, on the orch's instruction after the blocker.
- The scratch checkout and the recorded-run extracts stayed outside the repository.
- The way forward is the operator's: a per-run advisor option on `lym shuttle run`, or another route.
