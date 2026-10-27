# Bench the tuned forked Codex Astra fan on P1 at medium effort

Adds `docs/research/codex-astra-tuned-p1-med-bench-2026-10-27/` with a `report.md` and its `data/`.
No production code, `PATTERN.md` or module doc changes.

## What the bench measured

One `codex exec` handler at `gpt-6-astra`, medium effort, reviewed the `P1` plan (`gate-runs-once`, 33 cards) and spawned three lens forks (`affected-set`, `guard-reach`, `way-forward`, the lenses the Claude fan's round 1 used) with `fork_turns="all"`.
The context, the `AGENTS.md` and the tuned reading block are the tuned-prompt bench's, unchanged.
Only the input and the effort differ from that bench's `P4` run.

Result: the handler returned `BLOCKING` with 9 findings, 4 shared with the Claude fan (3 of its 12 findings), 5 `unique-real`, none `false`; neither of the fan's 2 `BLOCKING` findings was found.
The run took 414 s and 263k fresh input tokens; the weekly window read 18 percent at the first request and 27 at the end, on an account that sibling benches may have used at the same time.
The rollouts show three overlapping forks that inherited the handler's context, and an `AGENTS.md` loaded in every rollout.

## Design decisions

- The harness is the tuned-prompt bench's copied with only `ROOT`, the block path and the runner's effort changed, so the block, the inlined PATTERN overview and `AGENTS.md` are produced by the same scripts; `tuned-block.md` is byte-identical.
- The forks' final messages were taken from the `FINAL_ANSWER` inputs in the handler's rollout, since the handler run left no lens files.
- The fan's `F6` counts as not found: the handler's `F2` sits in the same sentence of card 19 but names a different defect, and the grading says so.
- The handler's dropped fork finding (reroute-count and batten episodes at a skip) is graded `unique-real` on decision record line 146, against the handler's own reason for dropping it.
- Findings graded `unique-real` hold on reading the cited card lines and the snapshot's code; none was run, and the report says so.

## Review notes

The change is documentation and shell scripts under `docs/research/`; no test or build reads it.
`data/harness/` works on `.scratch/bench/`, which is not committed.
