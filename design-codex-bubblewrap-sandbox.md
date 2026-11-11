# Codex keeps forcing its bubblewrap sandbox, so astra fails on OSL-1046

`codex-bubblewrap-sandbox` · Task · bug, codex, shuttle · draft

## Seen

On 2026-11-09 the operator reported that Codex has so far been mostly trouble in lym: it insists on running under its bubblewrap sandbox, which gets in the way of the work lym gives it.
The `astra` model (`gpt-6-astra` in `models.yaml`) fails on OSL-1046 for that reason.
`testmap-verb`'s first Code-Review round recorded its astra assist only as `assist_ending: stopped`, with no error.
The operator then had astra removed from every task's `run_settings`.

## Idea

Find what makes Codex choose bubblewrap in a lym session and settle the sandbox mode lym launches it with, so a Codex seat runs on OSL-1046 like a Claude one.
lym reports a Codex seat that cannot start, naming the cause, instead of ending it quietly as stopped.