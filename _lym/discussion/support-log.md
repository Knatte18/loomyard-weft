## Interview

A darn run holds no interview; the scope comes from the board entry and a coarse reading of the forked fan bench's report and harness.

## Rejected alternatives

- Hand-writing the PATTERN block into the prompt: rejected, because it would drift from what production renders; the staging script fills the production stencil instead.
- Passing `CLAUDE.md` text in the prompt instead of `AGENTS.md`: rejected, because the task asks for Codex to load it as a Claude session loads `CLAUDE.md`.
- Re-running the harness unchanged under the sibling bench's paths: rejected, because the copied scripts name the forked fan bench's worktree and would write outside this one.
- A repeat run to measure variance: rejected by the task's cap of one run.

## Review rounds

_No rounds yet._

## Question ledger

- Resolved (self-pick): which date names the output directory. The run date, 2026-10-27.
- Resolved (self-pick): which PATTERN text the directive inlines. The snapshot's own `PATTERN.md`, since the Claude round reviewed against the code commit's tree.
- Open: whether Codex loads `AGENTS.md` outside a git repository. Answered by the rollout during the run.
