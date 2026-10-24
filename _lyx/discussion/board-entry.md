# Board entry `batch-trial-script-research`: Batcher wide-trial comparison script and results into docs/research

## Task (operator, 2026-10-24)

Put the comparison script and results of audit `batch-budget-300k-trial` into git, so the check can be rerun after more `wide` runs land.

- Source, machine-local in the prime: `/home/knatte/Code/loomyard-LYXHUB/loomyard/.scratch/a7/compare.sh` and its outputs `summary.md`, `runs.tsv` and `forks.tsv`. Read them from there; leave that folder as it is.
- Target: `docs/research/batch-budget-trial-2026-10-24/`, holding `compare.sh`, a `README.md` (what the script compares, how to run it, its inputs: the records repo's archive tags and the local Claude Code transcripts through `tools/tokencount`, and the 2026-10-24 result from `summary.md`), and the 2026-10-24 `runs.tsv` and `forks.tsv` as the measured baseline.
- Adapt `compare.sh` so it runs from its new place: the repo root found from the script's location, its cache, built `tokencount` binary and outputs written under the repo's `.scratch/batch-budget-trial/` by default, never into `docs/`.
- No cached `tokencount` reports, no binary and no logs go into git.
- The run changes no code outside the new folder.
