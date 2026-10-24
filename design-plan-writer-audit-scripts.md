# Keep the Plan writer audit scripts and baseline in docs/research

`plan-writer-audit-scripts` · Task · audit, loom · done

## Task (operator, 2026-10-24)

Put the scripts and tables of audit `plan-writer-context` (A8) into git, so its measurements can be rerun once more loom runs land.

- Source, machine-local in the prime: `/home/knatte/Code/loomyard-LYXHUB/loomyard/.scratch/a8/`. Read it from there; leave that folder as it is.
- Target: `docs/research/plan-writer-audit-2026-10-24/`.
- Keep: the scripts `run-all.sh`, `run-priority.sh`, `reads.sh`, `q1-table.sh`, `q1-summary.sh`, `q2-table.sh`; the inputs `slugs.txt` and `sample-input.tsv`; the 2026-10-24 result `results.md` and its tables `q1-table.tsv`, `q1-summary.tsv`, `q2-table.tsv`, `reads-summary.tsv`, `timing-plan-phase.tsv` as the measured baseline.
- Add a `README.md`: what each script measures, the order to run them, their inputs (local Claude Code transcripts through `tools/tokencount`, `lyx report timing`, the records repo), and a pointer to `results.md`.
- Adapt the scripts so they run from their new place: the repo root found from the script's location, the built `tokencount` binary, per-run reports and outputs written under the repo's `.scratch/plan-writer-audit/` by default, never into `docs/`.
- No binary (`tokencount-bin`), no `.bak`, no logs, no `.err` files and no per-run raw dumps (`tc/`, `reads/`, `t1`, `t2`, `tags*.txt`, `slugs-*.txt`, `extra-loom.tsv`, `slug-tag.tsv`, `timing-since-*.md`, `tc-lenses-language-sets.md`) go into git.
- The run changes no code outside the new folder.
