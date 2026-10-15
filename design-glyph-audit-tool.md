# Webster fork read audit as a tool under tools/

`glyph-audit-tool` · Note · enhancement, webster

## Seen

The Webster fork read audit of 2026-10-13 (board note `glyph-use-audit`) ran on jq scripts: `extract2.jq` pulls per-fork rows from Webster implementer fork transcripts (pack parsing, edits, first-turn reads), `stats-base.jq`, `stats2.jq` and `stats3.jq` aggregate them, and by-hand steps build `files.txt`, `table.tsv` and `turns.tsv`.
Since `290c773d0` the scripts, the report and a README with the re-run recipe sit in `docs/research/glyph-audit-2026-10-13/`.
The README names the steps that were never recorded: how `files.txt` was built, the exact `extract2.jq` invocation, the script that produced `turns.tsv`, and the commands behind the pack-quality figures.

Operator view (2026-10-15), not a decision: scripts like these must be kept in the repository, not in scratch.

## Idea

A repeatable tool under `tools/`, like `tools/tokencount/tokencount.sh`: one script that takes a date range or a list of runs, finds the Webster implementer fork transcripts, and reports per fork whether it read its pack's spans in one parallel first work turn, plus exploration calls per card, split by pack on or off and, once `webster-pack-layout-switch` lands, by pack layout from the begin-batch trace line.
