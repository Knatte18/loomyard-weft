# Orch cycle mode as config: clear or compact

`orch-cycle-compact` · Notes · feature

Seen 2026-10-03: the orch's context cycle uses `/clear` plus a handoff file, which starts a new conversation and may drop a Remote Control link the operator relies on from a phone.
Cycling through `/compact` keeps the same session, so Remote Control and session identity survive.
Idea (operator): the cycle mode as a config option, `clear` or `compact`; weigh what the handoff file gives that a compaction summary does not.
