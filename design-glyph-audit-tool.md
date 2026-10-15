# Webster fork read audit as a tool under tools/

`glyph-audit-tool` · Task · enhancement, webster · running · Webster-Burler-Fix

## Seen

Two Webster fork read audits ran on jq and shell scripts:

- 2026-10-13, kept in `docs/research/glyph-audit-2026-10-13/` (`290c773d0`): pack on versus off and read-all-first, with a README naming the steps that were never recorded.
- 2026-10-15, kept in `docs/research/glyph-audit-2026-10-15/`: the pack-file layout (`1e39afc10`) against read-all-first, pack correctness at batch start (spans checked against the batch's `startSha` tree from the records repo's `state.json`), missed files, and plan writers' glyph use; its `COMMANDS.md` records every command in order.

Operator view (2026-10-15), not a decision: scripts like these must be kept in the repository, not in scratch.

## Scope

One repeatable tool under `tools/`, like `tools/tokencount/tokencount.sh`, built from both audits' scripts, so the next pack layout (task `kickstart-per-card`) can be measured against today's with one command.

- Input: a date range or a list of runs; it finds the Webster implementer fork transcripts itself (the `find` in the 2026-10-15 `COMMANDS.md`).
- It classifies each fork's pack layout from the prompt and, where present, from the `webster: kickstart pack` begin-batch trace line, so arms are not hand-built.
- Per fork and per arm: whether the pack was opened and when, share of spans read before the first edit and in the first work turn, first-turn parallelism, calls before the first edit, exploration calls per card, reads, greps, quarry lookups, turns, output tokens and cache reads per card, and edits to files with no span.
- Optional sections: pack correctness against the batch's start tree, and plan glyph use (glyphs, `plan:` handles, bare paths per card; plan writers' quarry calls against greps and reads).
- Output: a markdown report, medians and pooled per card, with sample sizes per arm.
- Shell and jq only, no Python; it follows PATTERN and its scans like any code under `tools/`.
- The two `docs/research/` directories stay as the record of their runs; the tool does not rewrite them.

## Run settings (orch, on the operator's word 2026-10-15)

This task is the single-run trial of the Go driver (`driver_kind: go`, with the `haiku` tackler and loom's arbiter), with the low review budget.
