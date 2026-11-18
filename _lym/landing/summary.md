# Estimate the size of a clean rewrite of seven patched modules

Adds a research report under `docs/research/rewrite/shrink-estimate-2026-11-18/` and an index entry in `docs/research/README.md`.
No code or test changes; the only runs were `codestats` and `dupl`.
It is the input to the clean-room rewrite note `clean-room-module-rewrite`.

## What the report holds

- **Measured, all seven modules** (`loomcli`, `fabricengine`, `landingshed`, `shuttleengine`, `battencli`, `websterengine`, `driftengine`): production and test lines split into code, comment and blank, and duplicate code from `dupl@v1.0.0` at 50 tokens, over code and over tests.
  `measure.sh` and `union.sh` reproduce the table.
- **`landingshed-spec.md`**: what `landingshed` must do and why, as eleven invariants and a rule table per row, each rule tied to the commit that created it.
  It is written to be step 1 of a pilot.
- **Clean-size estimate for `landingshed`**: a parts table with a line estimate each, what today's code would drop, and the test lines a spec-driven suite needs.
- **Extrapolation to the other six modules**, labelled as one, from the measured numbers and the `landingshed` ratios.

## Findings

- Duplicate code is small: 850 of 126,245 production lines (0.7%) and 2,919 of 192,868 test lines (1.5%) across the seven modules.
  The largest cross-package clone is 58 lines, between the Claude and Codex engines in `shuttleengine`.
- The size is in comments (27.4% of production lines, 19.8% to 31.1% per module) and tests (1.94 test code lines per production code line, 1.29 to 2.65 per module).
- `landingshed` clean: about 4,400 code lines against 5,418 (range 4,000 to 4,750), about 900 to 1,400 comment lines against 2,568, and about 4,000 to 6,000 test lines against 12,585.
- All seven modules, extrapolated: 319,113 lines today, 142,048 to 196,569 after, so 38% to 55% fewer.
  Tests account for 74% to 80% of the lines removed; production code itself shrinks 12% to 26%.

## Design decisions

- The code-side sizes come from one full read of the `landingshed` production files by a subagent, so they are a judgement, not a tool count.
  The test figure counts spec behaviours and is the least certain number; a pilot would measure it.
- The extrapolation assumes the other six modules behave like `landingshed`; none was read beyond the mechanical counts.
  The report names the modules where that is least safe.
- Per-rule commit citations in the spec come from commit subjects and bodies, not from each diff, and a reason with no commit is an inference.
- `dupl` at 50 tokens misses near-duplicates that differ in names or types, which the code read found; a lower threshold was outside the brief.
