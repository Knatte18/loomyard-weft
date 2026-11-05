# Live trial: creel mail reaches Codex sessions through the new hooks

`codex-creel-trial` · Task · research, codex, creel · running · Worktree-Create

Depends on: [`codex-creel-hooks`](design-codex-creel-hooks.md)

## Goal

A live check, on real Codex sessions in the sandbox hub `lyt`, that creel mail reaches a Codex strand through the hooks `codex-creel-hooks` adds, as it reaches a Claude strand.

## Steps

Launch Codex sessions in `lyt` through `lym shuttle` with the installed `lym` built from the landed `codex-creel-hooks`, and record for each case whether the mail arrives, how fast, and whether the model acts on it:

1. Mail posted while the session runs tools: delivered at the next tool boundary through `PostToolUse`.
2. Mail posted during the last step of a turn: delivered at turn end through `Stop`, and the session goes on to answer it.
3. Mail posted to an idle session: the daemon wakes it through `codex queue`.
4. A program's notice and an agent's mail that expects a reply: the session answers with `lym creel post --reply` from inside its `workspace-write` sandbox.
5. Several mails in a row and two Codex sessions at once, each getting only its own mail.

## Rules

- The trial runs everything; the operator does nothing in `lyt`.
- Findings go to a report under `docs/research/`, and each lym fault becomes a bug note in project `codex-engine-trial`.
