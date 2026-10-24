# Decision record — plan-writer-audit-scripts

## Goal

Archive the scripts, inputs and 2026-10-24 baseline of audit `plan-writer-context` (A8) under `docs/research/plan-writer-audit-2026-10-24/`, so its measurements can be rerun once more loom runs land.

## Scope

- New folder `docs/research/plan-writer-audit-2026-10-24/`, and nothing outside it.
- Scripts: `run-all.sh`, `run-priority.sh`, `reads.sh`, `q1-table.sh`, `q1-summary.sh`, `q2-table.sh`.
- Inputs: `slugs.txt`, `sample-input.tsv`.
- Baseline: `results.md`, `q1-table.tsv`, `q1-summary.tsv`, `q2-table.tsv`, `reads-summary.tsv`, `timing-plan-phase.tsv`.
- A new `README.md`: what each script measures, the run order, the inputs and a pointer to `results.md`.
- Source is `/home/knatte/Code/loomyard-LYXHUB/loomyard/.scratch/a8/`, read only and left as it is.
- Left out of git: the `tokencount-bin` binary, `.bak`, logs, `.err` files, and the per-run dumps (`tc/`, `reads/`, `t1`, `t2`, `tags*.txt`, `slugs-*.txt`, `extra-loom.tsv`, `slug-tag.tsv`, `timing-since-*.md`, `tc-lenses-language-sets.md`).

## Decisions

- Decision: every script finds the repo root from its own location, `REPO="$(cd "$HERE/../../.." && pwd)"`, and takes `WEFT` (default `$REPO/../loomyard-weft`) and `OUT` (default `$REPO/.scratch/plan-writer-audit`) from the environment.
  Rationale: the same convention as the neighbouring `batch-budget-trial-2026-10-24/compare.sh`; any working directory and any worktree works.
- Decision: the scripts build `tools/tokencount` into `$OUT/tokencount-bin` on first use instead of expecting a prebuilt binary.
  Rationale: the task asks for the built `tokencount` binary, and a run from a fresh checkout then needs no manual step.
- Decision: per-run reports (`tc/`, `reads/`) and generated tables (`q1-table.tsv`, `q1-summary.tsv`, `q2-table.tsv`, `slug-tag.tsv`, `slugs-*.txt`) are written under `$OUT`, never into the script's folder.
  Rationale: the committed tables stay the 2026-10-24 baseline; a rerun cannot overwrite them.
- Decision: the inputs `slugs.txt` and `sample-input.tsv` are read from the script's folder.
  Rationale: they are committed inputs, not outputs.
- Decision: `q1-table.sh` takes `extra-loom.tsv` from `$OUT` when it exists and skips it otherwise.
  Rationale: the file is excluded from git, so a fresh checkout has only the `runcost.tsv` loom rows; the bound is that a rerun then misses the archive tags `extra-loom.tsv` added.
- Decision: the scripts keep their logic unchanged; only paths, the cwd handling and the header comments change.
  Rationale: a rerun must measure what the baseline measured.
- Decision: the baseline files are copied byte for byte.
  Rationale: they record one run and are not claiming to stay true.
- Decision: `README.md` follows the layout of the `batch-budget-trial-2026-10-24` README.
  Rationale: the same kind of folder, so the same shape.

## Constraints

- PATTERN: `PATTERN-documentation-lifecycle` and the `.md` rule of one sentence per line apply to `README.md`.
- No code outside the new folder changes, so no `doc.go`, `docs/overview.md` or PATTERN edit is due.
- Scripts need `git`, `jq`, `awk` and `go` with `CGO_ENABLED=1` and a C compiler on `PATH`.

## Auto-mode assumptions

- The records repo is the sibling `loomyard-weft` of the repo's worktree in the hub, as for `compare.sh`.
- `docs/research/token-audit-2026-10-22/runcost.tsv` stays in the repo, since `q1-table.sh` reads it.

## Open risks

- The result tables are tied to the local Claude Code transcripts of one machine; a rerun elsewhere has fewer rows.
- `results.md` mentions the old prime-relative commands; it is kept unchanged as the baseline, and the README holds the current commands.

## Acceptance criteria

- The folder holds exactly the listed scripts, inputs, baseline files and `README.md`.
- `bash -n` passes on every script, and each script resolves its paths from its own location.
- No script writes into `docs/`; a grep of the scripts finds no `.scratch/a8` or `/home/knatte` path.
- Baseline files are identical to the source files.
- `README.md` names each script's measurement, the order `run-all.sh`, `run-priority.sh`, the timing report, `q1-table.sh`, `q1-summary.sh`, `reads.sh`, `q2-table.sh`, the inputs, and `results.md`.
- `git status` shows changes only under the new folder.
