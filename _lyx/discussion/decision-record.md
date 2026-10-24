## Goal

Keep the `batch-budget-300k-trial` comparison script and its 2026-10-24 measurements in git, so the cautious-versus-wide check reruns once more `wide` runs land.

## Scope

- New folder `docs/research/batch-budget-trial-2026-10-24/` holding `compare.sh`, `README.md`, `runs.tsv` and `forks.tsv`.
- `compare.sh` is the machine-local script from the prime's `.scratch/a7/`, adapted to run from its new place.
- `runs.tsv` and `forks.tsv` are copied unchanged as the measured baseline.
- `README.md` states what the script compares, how to run it, its inputs and the 2026-10-24 result from `summary.md`.
- No code outside the new folder changes.
- The prime's `.scratch/a7/` stays as it is; no cached `tokencount` report, binary or log is committed.

## Decisions

- **Repo root from the script's location.**
  `REPO` is derived as the script's grandparent-of-grandparent (`docs/research/<folder>/` up three levels), replacing the `.scratch/a7` depth.
  Rationale: the script runs from any cwd and any worktree.
- **Work directory under `.scratch/batch-budget-trial/` by default.**
  `OUT` defaults to `$REPO/.scratch/batch-budget-trial`, overridable by an `OUT` environment variable; the cache `tc/`, the built `tokencount-bin`, `runs.tsv`, `forks.tsv` and `summary.md` are written there, never into `docs/`.
  Rationale: `.scratch/` is git-ignored, so a rerun leaves the committed baseline untouched.
- **Records repo stays `WEFT`, default `$REPO/../loomyard-weft`.**
  Rationale: unchanged behavior; the README names the override for a worktree whose sibling differs.
- **Baseline files are verbatim copies.**
  Rationale: they are the measurement; the README carries the summary table.
- **README is a measurement report for the table, a how-to for the rest.**
  The table is stated as one run's result, not a claim to stay true.

## Constraints

- `PATTERN-comment-line-breaks` and the markdown rule: one sentence per line in `README.md`; table cells stay on one line.
- No tracked path under `.scratch/`; `compare.sh` keeps its executable bit.
- The task edits no Go code, so the `tools/` scans see nothing new.

## Auto-mode assumptions

- The prime's `.scratch/a7/summary.md` is the 2026-10-24 result; its table is copied into the README as it stands.
- The script's logic beyond its paths is kept as is.

## Open risks

- A rerun needs the records repo's archive tags and the local Claude Code transcripts; on another machine `wide` rows lack measured peaks and costs (`NA`).
- The script is not run end to end here, because the rerun is long; a syntax check and a read of every path change stand in.

## Acceptance criteria

- `bash -n docs/research/batch-budget-trial-2026-10-24/compare.sh` passes.
- The script holds no path into `docs/` for output and no `.scratch/a7` reference.
- `git status` after the change lists only the four new files.
- `runs.tsv` and `forks.tsv` are byte-identical to the prime's.
