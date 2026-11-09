# Codex keeps forcing its bubblewrap sandbox in lym runs

`codex-bubblewrap-sandbox` · Note · bug, codex, shuttle

## Seen

On 2026-11-09 the operator reported that Codex has so far been mostly trouble in lym: it insists on running under its bubblewrap sandbox, which gets in the way of the work lym gives it.

## Idea

Find what makes Codex choose bubblewrap in a lym session and settle the sandbox mode lym launches it with, so a Codex seat runs like a Claude one.
The details of each failure still need collecting from the runs that hit it.