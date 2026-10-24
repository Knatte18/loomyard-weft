# Keep the batcher budget trial script and baseline in docs/research

Adds `docs/research/batch-budget-trial-2026-10-24/`, so the cautious-versus-wide batcher check of audit `batch-budget-300k-trial` can be rerun once more `wide` runs land.
No code outside the new folder changes.

## What the folder holds

- `compare.sh` groups Webster runs from the records repo's archive tags by the profile their own `state.json` names, and prices them from the local Claude Code transcripts through `tools/tokencount`.
- `README.md` states what the script compares, how to run it, its inputs and the 2026-10-24 result.
- `runs.tsv` and `forks.tsv` are the 2026-10-24 outputs, copied unchanged as the measured baseline.

## Changes to the script

- The repo root is found from the script's location, three levels up, so it runs from any directory and any worktree.
- The cache, the built `tokencount` binary and the outputs go under `.scratch/batch-budget-trial/` by default, which is git-ignored; an `OUT` environment variable overrides it.
- The header comment names the new paths; the comparison logic is unchanged.

## Design decisions

- The baseline files are verbatim copies, and the summary table lives in the README, so `summary.md` is not committed.
- The script was syntax-checked with `bash -n` but not rerun end to end: a run re-reads every archive tag and transcript.
- No cached `tokencount` report, binary or log is committed.
