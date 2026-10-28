# Bench the decision record as yardstick for the forked Codex reviewer

Adds `docs/research/codex-review-yardstick-2026-10-28/`, a follow-up to the Codex code-review bench of the same day.
It changes no code.

The earlier bench's forked Codex fan rejected the Claude fan's decision-record findings because the rendered rubric and fasit text call the record the writer's own claim and the board snapshot the only measuring stick.
This bench adds one bullet to `data/prompts/block-v2-code.md` naming the record's decisions and acceptance criteria as a yardstick beside the snapshot, overriding that reading, and runs the fan once per input (two `codex exec` handler runs, `gpt-6-astra` at medium).
Everything else is the earlier bench's: the harness, copied with its root path changed, the two inputs and reviewed commits, the reference, context parity and the fork cap of 20.

Result, one run per input: all 5 of the Claude fan's findings found on `spawn-text-file-busy` (2 of 5 in each earlier run), the same 3 of 6 on `fixer-overcap-review`, and no new false finding on either.
The report holds the comparison with the earlier runs, the per-finding table for the record findings, and its limits (one run per input, one grader).

## Design decisions

- The line states its override of the rubric's "writer's own claim" text and says a divergence is a finding whether or not the board entry names it, because the earlier runs rejected a finding on the entry naming no spelling.
- The earlier bench's files stay unchanged; the harness is copied under the new directory, and the reference findings are read from the earlier bench's `data/reference/` instead of being copied.
- Both runs ran in parallel on one account, as the earlier pairs did; none was rerun.
- Scratch trees and `CODEX_HOME` directories under `.scratch/` were removed after the runs.
