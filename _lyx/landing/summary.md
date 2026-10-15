# tools/forkaudit: repeatable Webster fork read audit

Adds `tools/forkaudit/`, one shell-and-jq command that measures how Webster implementer forks read their kick-start pack, per pack layout.
It replaces the hand-run jq and shell pipelines of the 2026-10-13 and 2026-10-15 audits, so the next layout (`kickstart-per-card`) can be compared with today's in one run.
`docs/research/glyph-audit-2026-10-13/` and `docs/research/glyph-audit-2026-10-15/` are untouched.

## What it does

`forkaudit.sh` takes `--since <date>` (with optional `--until`) or `--runs <name>[,<name>...]`, finds the fork transcripts under the projects directory, and writes a markdown report to `.scratch/fork-audit.md` (`--out` overrides).
The report holds a samples table, pack-use and arm-comparison tables with medians and pooled values and per-cell sample sizes, exploration quartiles, and a per-fork table.
Two flags add sections:

- `--pack-check` checks each pack span against the file at the batch's `startSha` and lists files the pack gave no span for.
- `--plan` counts glyph, `plan:` handle, bare-file and bare-directory references per plan card, and the plan writers' quarry calls against greps and reads.

Intermediate files stay in `.scratch/fork-audit/` (`--work` overrides) for drill-down.

## Design decisions

- The prompt decides each fork's arm (`file`, `raf`, `inline`, `none`, `unknown`) from one marker table at the head of `extract.jq`; a new layout adds one row.
  The `webster: kickstart pack` trace line only cross-checks the prompt, since it cannot tell `raf` from `inline`.
  A trace value the table does not know, on an `unknown` fork, makes the arm `trace:<value>`, so a new layout is measured as its own arm.
- The call classifier and span-hit rule are carried over unchanged from the 2026-10-15 `extract-pf.jq`, so numbers stay comparable with both kept audits.
- Transcript lines parse one by one; an unparsable line is dropped and counted in the header, so a fork still running stays measurable.
- Batch resolution matches `spawnedAt` within 1 to 6 seconds of the fork's first line, searched only on the fork's own run's refs (`<run>-weft` and `archive/<run>/*`), so concurrent runs never cross-resolve.
- The work directory's clearing is bounded by a `.forkaudit-work` marker and canonical-path checks; the report is written through a `mktemp` file beside `--out` and moved into place on success only.
  Git use is read-only (`log`, `show`, `ls-tree`, `cat-file`).
- `PATTERN.md`: the `PATTERN-lyxdirs-single-declarer` line now admits one variable block heading a `tools/` script, which is where `forkaudit.sh` spells its `_lyx` and `.lyx` literals.
- Tests are Go integration tests (`//go:build integration`) that run the script on synthetic fixtures under `testdata/`, with the `TestMain` shape of `tools/tokencount`; `tools/forkaudit` gets its row in `cmd/lyx/testdata/test-budget.yaml`.
  They need `jq` on non-Windows hosts and skip on Windows.

## Check by hand

- Two review findings were left open: the report-identity assertion in the tests when a records path is appended to the arguments (`tools/forkaudit/forkaudit_integration_test.go`), and the `.jq` filters spelling directory roots outside the script's variable block (`tools/forkaudit/*.jq`, against `PATTERN-lyxdirs-single-declarer`).
- Run `forkaudit.sh` against real transcripts, with and without `--pack-check` and `--plan`; only fixtures have exercised it.
- The batch join by `spawnedAt` is heuristic; a slow fork start leaves a batch unresolved, which the report counts as `notree`.
- Not run: `lyx gate test ./...` and `lyx gate test --tags integration ./...` before pushing to `main`, as `CLAUDE.md` requires.
