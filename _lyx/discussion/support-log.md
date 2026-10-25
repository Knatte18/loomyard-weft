# Support log — Bench Codex Astra against solo Opus as a plan reviewer

## Interview

A darn run holds no interview; the board entry is the whole statement.

## Rejected alternatives

- Run Opus through `claude -p` with the earlier harness's shell loop.
  Rejected: agents run in interactive tmux sessions only.
- Run Opus through a Claude Code subagent from this session.
  Rejected: the Agent tool is unavailable here, and a subagent would inherit this session's context, which breaks the like-for-like prompt.
- Rebuild the inputs from `main` at the round's start time.
  Rejected: the earlier bench recorded each input's code commit; rebuilding could pick a different snapshot.
- Add a second Astra run per plan to estimate noise.
  Rejected: the entry caps the bench at four runs.

## Review rounds

_No rounds yet._

## Question ledger

- Q: How does Opus solo run without `claude -p`? Resolved (self-pick): `lyx shuttle run --model opus --effort medium` with the input's `prompt.md` as `--prompt-file`.
- Q: Where does the report date come from? Resolved (self-pick): the entry's date, 2026-10-25, matching the entry's Goal line.
- Q: What if the Plus window is already high before the first run? Resolved (self-pick): stop the Codex cells and report, per the entry.
