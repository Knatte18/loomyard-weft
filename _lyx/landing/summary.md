# Research: Codex twill-like fork fan spike report

Adds `docs/research/codex-twill-spike-2026-10-24/report.md`, the report of a second hands-on Codex CLI spike (0.164.0) that asks whether twill, a fan of forks inheriting the parent's conversation, can run on Codex.
Nothing is built and nothing outside the new folder changes.

## What the report covers

- Parallelism: three children run at once by default (the parent is told it has four slots, itself included); a fourth concurrent spawn fails with `agent thread limit reached`, and `-c agents.max_concurrent_threads_per_session=8` lifted the cap for an eight-child fan.
- The wait: `wait_agent` names no agent; each child's result reaches the parent as a `FINAL_ANSWER` `agent_message` item.
- `fork_turns`: across 81 `spawn_agent` calls in 20 sessions, a named value was always honored and a silent prompt always got the tool's default `all`; the failure seen was a fan that was never spawned (2 of 20 sessions).
- Cost: a forked child's first call reads the parent's context plus about 7.5k tokens; tables give per-child and per-fan totals with and without a 14k-token file in the parent's context.
- Audit without hooks: what a child's rollout, the parent's rollout and the `PreToolUse` hook show, including that `PreToolUse` does fire for children (parent `session_id`, child `transcript_path`), which corrects the first spike's "no hook fires for a subagent".
- The closing sections list what twill's directive would need to say on Codex and what shapes or blocks a port.

## Design decisions

- The spike ran Codex only in a private tmux server with a private `CODEX_HOME` under `.scratch/` holding a trust entry and an `auth.json` symlink, so `~/.codex/config.toml`, `~/.codex/rules/` and `~/.codex/sessions/` stayed untouched; their modification times were checked at the end.
- Counts come from rollouts, not from the model's replies: the secret-number probe proved weak (a message can carry the secret), so a second probe gave the codeword in an earlier turn and barred it from the task message.
- An earlier pass of ten sessions is left out of the counts because overlapping runs polluted the shared output folder; the report says so.
- The throwaway repository, private `CODEX_HOME` and rollouts remain under the gitignored `.scratch/twill/` because the shell refused the recursive remove; they hold account ids and are not tracked.
