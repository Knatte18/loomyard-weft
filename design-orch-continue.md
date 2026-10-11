# lyx orch start --continue resumes the last orch session

`orch-continue` · Note · feature, orch

Split out of note `orch-owns-the-loop` (2026-10-11).

After a reboot the operator wants the same conversation back, as `claude --continue` gives, not a fresh session reading a handoff.
Today that takes `lyx orch start --adopt <session-id>`, with the id dug out of `~/.claude/projects/`.
`lyx orch` records its session id and offers `lyx orch start --continue` (or makes it the default for a dead strand) to adopt the last orch session.
