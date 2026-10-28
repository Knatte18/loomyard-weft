# Bench the forked Codex Astra fan as Code reviewer on two darn diffs

Adds `docs/research/codex-code-review-bench-2026-10-28/`: the report, the adapted harness and the raw outputs of four `codex exec` handler runs.
Nothing under `internal/`, stencils or recipes changes.

The question is whether the forked Codex Astra fan can stand in for the Claude Code-Review fan on a darn diff.
The two inputs are the darn runs `spawn-text-file-busy` and `fixer-overcap-review`, which landed on `main` on 2026-10-28 with real Go diffs and Claude findings to grade against, in different packages.
Each was reviewed twice by `gpt-6-astra` at medium effort, handler and forks, with the three Code-Review lenses the Claude fan ran.

Result: the Codex fan found 2 of 5 reference findings on the first input and 3 of 6 on the second, the same ones in both runs of each, and missed the fan's one `BLOCKING` finding on the first.
Every shared finding is a missing test row; the divergences from the decision record that the Claude fan raised were not found.
The Codex fan also raised real findings the Claude fan lacks on the second input (two `PATTERN` readings and a parser gap), and false ones on the first.
The report records per run the figures the entry asks for, and the limits of two inputs and one grader.

## Design decisions

- Inputs: `spawn-text-file-busy` (verdict `BLOCKING`) and `fixer-overcap-review` (`APPROVED`), chosen from the day's landed darn runs; `dobby-task-limit-hold`, `usage-limit-wake` and `shuttle-hook-signals` were passed over for a thin reference, test-only findings and two rounds.
- The reviewed code is the task branch tip the Claude round started on (`7615d129`, `d50e5277`), recovered from the hub repository's unreachable task-branch history; the squash that landed on `main` includes the round's fixes and is not what was reviewed.
  The snapshot is a shared clone so the rubric's `git diff <base>..HEAD` works as it did for the Claude reviewer.
- Block v2 is reworded for a diff, not dropped line by line: one line with no code analogue is dropped and five are kept with the plan nouns swapped.
  This departs from the entry's "dropped" and is named in the report.
- Tool use is read-only and the run's `PATH` holds no `lym` or `go`, as in the plan benches, where production's Code-Review rows drive the real substrate; the repository's gate rule bars a raw test run and the snapshot builds under cgo.
- The reference is round 1 of a `sonnet` medium fan; the Codex fan differs from it in model, fork cap and reading, so the report states its result for the pair of setups.
- Unique Codex findings are graded real or false by reading the snapshot's code and rules; two judgment calls (the `Warn` per ETXTBSY retry, counted false, and the gate-parity finding, counted real) are named as such.
- The scratch tree, the four scratch `CODEX_HOME` directories and the staged snapshots were removed after the runs; the reviews, lens reports, prompts and rollout figures stay under `data/`.

## Verification

- `lym gate test --wait` is green on the commit.
- Exactly four handler runs were made, each exiting 0, with no retry; `data/rollout-evidence.txt` shows three `spawn_agent` forks per handler.
