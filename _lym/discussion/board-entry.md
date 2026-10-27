# Board entry `codex-astra-fork-fan-bench`: Bench a Codex Astra fan with real forks at high effort on one plan

One codex exec handler spawning 3 lens forks with fork_turns=all, Astra high, on P4

## Question

Does a real Codex Astra lens fan, with forks that inherit the handler's context, find what Claude's lens fan finds on a plan review?
`docs/research/codex-astra-fan-bench-2026-10-27/report.md` ran each lens as a separate cold `codex exec`, not as forks, at medium effort: it found 0 of the Claude fan's 11 findings on `P4`, and took 907 s with the calls in sequence.
This bench uses Codex's own forks at high effort.

## Cap

- One input: `P4`, the `machine-handover` plan, with the snapshot, prompts and grading of the earlier bench.
- One model and effort: `gpt-6-astra` at high effort, for the handler and its forks.
- One fan run: a single `codex exec` handler that spawns 3 lens forks in parallel, one run, no repeat.
- No Claude run: the Claude fan reference is the one the earlier benches recorded.
- Stop and report what was done if a rate-limit window passes 70 percent.

## Method

- Reuse the earlier fan bench's harness under its `data/harness/`, its `P4` snapshot and its 3 lenses.
- The handler is the round-1 reviewer template with the fan cluster rules and its spawn paragraph kept: it does the explore step once, then spawns one fork per lens through Codex's `spawn_agent` tool with `fork_turns="all"`, so each fork inherits the handler's reading, and the forks run in parallel.
  The fork prompt keeps its inherited-context sentence and its tool-call cap, as Claude's forks have them.
- Enable whichever of Codex's multi-agent features `spawn_agent` with `fork_turns` needs (`multi_agent`, or `multi_agent_v2` through `-c features.multi_agent_v2=true`), and record which.
- First confirm from the rollout that 3 forks started, ran concurrently and received the handler's context.
  If they did not, stop and report what the rollout shows; never fall back to separate cold calls.
- Record the wall time, the handler's and each fork's fresh input and output tokens, and the windows.

## Grading

Grade the handler's findings, and any fork finding the handler dropped, against the fan reference of `P4` as the earlier benches did: `shared`, `unique-real` or `false`, in `data/match.tsv`.
Report beside the earlier Astra fan, solo Astra, solo Opus and Claude fan rows.

## Output

`docs/research/codex-astra-fork-fan-bench-<date>/report.md` with its `data/`, landed as the run's commit.
