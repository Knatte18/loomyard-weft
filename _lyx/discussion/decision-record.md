# Decision record — codex-spike

## Goal

Drive Codex CLI 0.164 by hand in tmux and write `docs/research/codex-spike-2026-10-24/report.md`, the evidence a later `codexengine` is built from.
The report covers startup, trust, idle and busy pane readings, bracketed paste, Esc interrupt, hook payloads, the session file with `token_count` events, model and effort selection, approval and sandbox behavior, `AGENTS.md`, skills and fork inheritance.
It ends with the launch line and pane classifiers a `codexengine` would use.

## Scope

- In: one new folder, `docs/research/codex-spike-2026-10-24/`, holding `report.md` and any trimmed capture or payload sample files it links.
- In: throwaway experiments under `.scratch/` in this worktree, which is git-ignored.
- Out: any code, config, PATTERN or doc change outside the new folder.
- Out: edits to `~/.codex/config.toml` or `~/.codex/rules/`.
- Out: any Codex session in the `loomyard` tmux session.

## Decisions

- Codex runs only in the private tmux server `tmux -L codex-spike`, in a throwaway git directory under `.scratch/`.
  Rationale: the operator's sessions and the `loomyard` tmux server stay untouched.
- Every launch carries `-a never -s workspace-write` and `--no-alt-screen`.
  Rationale: the board entry names the approval and sandbox switches, and the feasibility study names the pane mode.
  Bound: a launch that must exercise a prompt (PermissionRequest, trust) drops `-a never` for that one run, in the throwaway directory only.
- Hooks are captured through per-run `-c` overrides that point each event at a script writing its stdin to a file under `.scratch/`.
  Rationale: nothing in `~/.codex` changes.
- Prompts to Codex are one short line each, on the cheapest model and lowest effort that the test allows.
  Rationale: the spike spends little of the operator's quota.
- Each report item states what was seen, with a capture trimmed to what a classifier needs; an item that could not be tested says so and why.
- The report ends with a proposed launch line and a classifier table (startup, trust, idle, busy, paste placeholder, interrupt), each backed by a capture.

## Constraints

- The run changes no code outside the new folder.
- Markdown follows semantic line breaks, one sentence per line, with table cells on one line.
- Report files carry no secrets: auth tokens, account identifiers and the operator's email are trimmed from every sample.
- `PATTERN.md` raises no entry on a report-only change.

## Auto-mode assumptions

- A feature that needs a second login or an API key is recorded as untested, never worked around.
- Billing terms are not browsed; the report restates the feasibility study's finding only if a test touches it.

## Open risks

- Codex's TUI text may change between versions; the report pins 0.164.0.
- A test may spend more quota than planned; each run stays at one or two tiny turns.
- Hook trust may refuse unhashed hooks; the report records what a bypass flag or a trust step needs.

## Acceptance criteria

- `report.md` has a section per item of the board entry's test list, each stating what was seen or why it was not tested.
- Pane captures show the startup, trust, idle, busy and paste readings, trimmed.
- Hook payload samples exist for SessionStart, Stop, UserPromptSubmit and PermissionRequest, or the report states which event never fired.
- The session file's `token_count` event is shown with its field names.
- The report states whether `-a never -s workspace-write` stops every prompt, and what it blocked among network and writes outside the workspace.
- The report states whether a subagent or `/fork` inherits the conversation, from a test.
- The report ends with the launch line and the classifier table.
- `git status` shows changes only under `docs/research/codex-spike-2026-10-24/` and the files lyx writes under `_lyx/`.
