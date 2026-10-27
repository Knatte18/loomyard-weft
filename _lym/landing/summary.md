# Bench Codex review block v2 on P4: one high run against two medium

Adds `docs/research/codex-v2-p4-bench-2026-10-27/` with its `report.md` and `data/`: a bench of Codex Astra's forked lens fan on plan `P4` with reading block v2 and a fork cap of 20 tool calls, one run at high effort and two at medium.
No product code, PATTERN entry or board entry changes.

## What it finds

Block v2 with the raised cap does not recover the Claude fan's missed findings.
The high run found 3 of the fan's 11 findings and 1 of its 3 `BLOCKING` ones; the two medium runs found 2 and 3, and 4 together, with 2 of 3 `BLOCKING` between them.
The union of the two medium runs found as many fan findings as the high run and more `BLOCKING` ones, for 20 percent more fresh input and half the wall time when run in parallel.
No run found the fan's `F1` (the `retryWeftPush` guard) or `F7` (the new leaf missing from `PATTERN-leaf-packages`); no run produced a `false` finding, and no handler dropped a fork finding.
The two medium runs, identical in every condition, share one fan finding and one of 11 distinct findings, which is the first repeat measure of these benches.

## Design decisions

- The harness is the earlier P4 medium bench's, with the path root moved to this worktree.
  Two changes carry the new conditions: `build-forkfan.sh` swaps in `block-v2.md` and changes only the cap sentence's number, stopping if either anchor is absent; `run-forkfan.sh` and `setup-codex-home.sh` take a label so each run has its own working tree and scratch `CODEX_HOME`.
- The three runs ran one after another, in the order high, medium 1, medium 2; the medium pair's parallel wall time is the longer of the two, not a measurement.
- Grading used the fan's round-1 review from the plan's weft commit, which the earlier benches had only as notes; the unmatched findings were checked against the snapshot only where the report says so.
- The weekly window peaked at 62 percent, under the 70 percent stop; sibling benches drew on the same account at the same time, so the report does not attribute window points to a run.
- The output directory is dated by the run, `2026-10-27`.

## Limits

One input, one run per cell and one grader: no setting's count separates from run-to-run variation, and the medium pair is the only repeat.
The fork prompts are encrypted in the rollouts, so that the forks saw block v2 and the cap of 20 rests on the script's anchor checks.
