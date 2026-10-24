# Keep the Plan writer audit scripts and baseline in docs/research

Audit `plan-writer-context` (A8) measured what the loom Plan phase costs and how much of the code the Plan writer read the Discussion's `relevant.md` named.
Its scripts lived only in the prime's `.scratch/a8/`, so the measurement could not be rerun once more loom runs landed.
This change archives them under `docs/research/plan-writer-audit-2026-10-24/`, with the 2026-10-24 result as the baseline.
No code outside the new folder changes.

## What the folder holds

- Scripts: `run-all.sh`, `run-priority.sh`, `reads.sh`, `q1-table.sh`, `q1-summary.sh`, `q2-table.sh`.
- Inputs: `slugs.txt` and `sample-input.tsv`.
- Baseline, copied unchanged: `results.md`, `q1-table.tsv`, `q1-summary.tsv`, `q2-table.tsv`, `reads-summary.tsv`, `timing-plan-phase.tsv`.
- `README.md`: what each script measures, the order to run them, the inputs (local Claude Code transcripts through `tools/tokencount`, `lyx report timing`, the records repo) and a pointer to `results.md`.

The `tokencount` binary, backups, logs, `.err` files and the per-run dumps (`tc/`, `reads/`, `slug-tag.tsv`, `extra-loom.tsv`, the timing report and the rest) stay out of git.

## Script changes

The scripts' logic is unchanged; a check of `q1-summary.sh` on the committed `q1-table.tsv` reproduces the committed `q1-summary.tsv`.
Only paths and headers changed, following the convention of `batch-budget-trial-2026-10-24/compare.sh`:

- Each script finds the repo root from its own location, so any working directory and any worktree works.
- `OUT` names the work directory and defaults to `<repo root>/.scratch/plan-writer-audit/`; reports, slug lists and generated tables go there, never into `docs/`.
- `WEFT` names the records repo and defaults to `<repo root>/../loomyard-weft`.
- `run-all.sh` and `run-priority.sh` build `tools/tokencount` into `$OUT/tokencount-bin` on first use.
- `reads.sh` derives the task worktree and the Claude Code transcript folder from the hub, the repo root's parent, instead of a fixed home path.

## Design decisions

- `q1-table.sh` reads `$OUT/extra-loom.tsv` only when it exists.
  The file is not committed, so a rerun on a fresh checkout lists the loom rows of `docs/research/token-audit-2026-10-22/runcost.tsv` alone and misses the archive tags the baseline added; the README says so.
- `results.md` keeps its prime-relative command list as the record of the 2026-10-24 run; the README holds the current commands.
- No docs, `doc.go` or PATTERN change is due: the change adds no module, CLI behavior or invariant.
- Only `bash -n` and the `q1-summary.sh` reproduction above were run; the other scripts need the local transcripts and the records repo and were not rerun.
