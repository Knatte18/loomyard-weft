## Interview

A darn run holds no interview; the board entry `codex-v2-p2-bench` is the whole statement of the change.

## Rejected alternatives

- Running the three handlers one after another: rejected, since the two mediums are to count as run in parallel and sequential runs would make the union's wall time a sum.
- Sharing one scratch `CODEX_HOME` across the runs: rejected, since the runner copies every rollout newer than its marker and would mix the runs' rollouts.
- Rebuilding the input from the weft and code commits by a new script: rejected, since the tuned-prompt bench's `build.sh`, `build-fan.sh` and `build-forkfan.sh` already stage `P2`'s kind and only their root path and the block and cap change.
- Re-running a run whose forks fail the evidence check: rejected, since the task says to stop and report.

## Review rounds

_No rounds yet._

## Question ledger

- Which lenses did the Claude fan's round 1 on `P2` use? Resolved: `affected-set`, `guard-reach` and `way-forward`, from `round-1-usage.yaml` at weft commit `394715f7f`.
- Where do the harness paths point? Resolved by self-pick: this worktree's `.scratch/`, see the first Decision.
- Start the three runs together or in sequence? Resolved by self-pick: together, bounded by the 70 percent window stop.
