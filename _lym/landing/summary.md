# Report how Codex takes hook-delivered creel posts

Adds `docs/research/codex-hook-delivery-2026-10-28/report.md`, the result of six Codex CLI runs on `gpt-6-luna` at `low` effort, each a reed strand with a hand-written hook script standing in for creel.
No Go code, stencil or PATTERN entry changes.

Findings per case:

- Idle wake: an async `Stop` hook exiting 2 does not wake an idle Codex session; `codex queue --thread <session_id> --message …` does, and the model acted on the message.
- `Stop` block: the reason is taken at once and acted on, and the operator sees `Blocked by hook` with it.
- `post_tool_use` block: taken at the next step and acted on when `AGENTS.md` carries the `[creel]` sentence; without it Codex reads the reason as a failed tool call and does not act.
- Folder trust: one accepted trust entry, keyed on the main checkout's root, covers a linked worktree of the same repository; the project hook ran there with no prompt.
- `Tab` queue: a line queued mid-turn reaches the model after the turn ends, not at the next step.
- `codex exec` runs the same `post_tool_use` and `Stop` hooks.

## Design decisions

- The spike used a `CODEX_HOME` of its own under the gitignored scratch area, seeded with a copy of `auth.json`; the copy and the home are deleted, and the operator's `~/.codex/config.toml` has an unchanged checksum.
- Case 1 reports two channels in one run, the async hook that failed to wake and the `codex queue` wake, since the board entry allows "a finding that none exists" and the queue verb is the working comparable.
- Case 5 used a throwaway repository and linked worktree under the scratch area with a project-level hook, because a first launch in an unknown repository is the only way to see the trust prompt; two launches made up the case.
- The runs are one sample each, as the board entry asks; the report states that limit.

## Not covered

- An untrusted folder with the same project hook, to show the hook is withheld.
- Whether a block or `additionalContext` from a hook reaches the model under `codex exec`.
- A hook calling `codex queue` itself, rather than a shell outside the session.
