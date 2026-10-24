# Decision record — codex-twill-spike

## Goal

Answer whether twill, a fan of forks that inherit the parent's context, can run on Codex.
The deliverable is one report, `docs/research/codex-twill-spike-2026-10-24/report.md`, built from a hands-on spike.

## Scope

- In: a throwaway git repository and a private `CODEX_HOME` under `.scratch/`, a private tmux server `codex-twill`, 3-4 parallel subagents spawned with `fork_turns: "all"`, each writing one file, the parent waiting and merging.
- In: measurements of parallelism, the wait, how reliably a prompt gets `fork_turns: "all"` over several tries, the token cost of an inherited context from the rollouts' `token_count` events, and what the subagents' rollouts show for auditing without hooks.
- In: a closing section on what twill's directive would need to say on Codex and what blocks it.
- Out: any build, any change outside the new report folder, any edit of `~/.codex/config.toml` or `~/.codex/rules/`.

## Decisions

- Decision: run Codex only in `tmux -L codex-twill`, never in the `loomyard` session, launched with `-a never -s workspace-write`, the cheapest model (`gpt-6-luna`, effort `low`) and one-line prompts.
  Rationale: the operator's task statement, and it matches the first spike's setup so the numbers compare.
- Decision: give Codex its own `CODEX_HOME` under `.scratch/`, with a `config.toml` holding a `[projects."<repo root>"]` `trust_level = "trusted"` table and a symlink to `~/.codex/auth.json`.
  Rationale: skips the trust prompt and keeps the operator's config and rules untouched; the isolated home also drops the operator's allow rules, so the sandbox holds.
- Decision: send prompts by bracketed paste followed by a separate Enter.
  Rationale: the first spike showed typed text plus Enter in one burst loses the Enter.
- Decision: the reliability test repeats the same fan prompt in fresh sessions and reads each spawn's `fork_turns` from the rollout's `spawn_agent` call, not from the model's reply.
  Rationale: the rollout is the record of what was actually sent.
- Decision: report measurements as observed in this run, with the rollout line types they come from, and state what was not tested.
  Rationale: a spike report records one run; the consumer is the future Codex engine design.

## Constraints

- The run changes nothing outside `docs/research/codex-twill-spike-2026-10-24/` and the gitignored `.scratch/`.
- No hooks are configured for the spike unless a test needs them; the audit question is answered from rollouts.
- Markdown follows semantic line breaks, one sentence per line.

## Auto-mode assumptions

- A darn run holds no interview; the scope is the board entry's `Task` section as written.
- The throwaway repository carries no lyx tooling; the fan's work is file writes of a few lines.

## Open risks

- Model behavior varies run to run, so a count of `fork_turns` picks over a few sessions is indicative, not statistical.
- The account's rate limits bound how many sessions the spike can run.
- A private `CODEX_HOME` may change defaults (status line, rules) against the first spike; the report notes any difference seen.

## Acceptance criteria

- The report exists at the named path and covers parallelism, the wait and completion signal, `fork_turns` reliability across several tries, per-subagent inherited-context cost, and rollout-based audit.
- The report ends with what twill's directive needs to say on Codex and what blocks it.
- Every number in the report traces to a rollout or pane reading from this run.
- The scratch folder and the tmux server are cleaned up, and no file outside the new folder differs in the diff.
