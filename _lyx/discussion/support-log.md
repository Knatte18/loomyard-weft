# Support log — codex-spike

## Interview

A darn run holds no interview; the scope comes from the board entry and a coarse reading of `docs/research/codex-feasibility.md` and the note `codex-engine`.

## Rejected alternatives

- Driving Codex through `codex exec`: rejected, since the engine under study is the interactive TUI in tmux.
- Using the shared `loomyard` tmux session: rejected by the board entry; a private server isolates the spike.
- Editing `~/.codex/config.toml` for hooks: rejected; per-run `-c` overrides leave the operator's config alone.

## Review rounds

_No rounds yet._

## Question ledger

- Q: Which model and effort for the cheap runs? Self-pick: the cheapest model the account lists, at the lowest effort; the report names what was used.
- Q: Does hook trust block unhashed `-c` hooks? Open; the first hook run answers it.
- Q: Are Codex model names the Sol, Luna and Astra aliases of the note? Open; the model-selection test answers it.
