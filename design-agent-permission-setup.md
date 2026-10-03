# Agents run with a deliberate tool and permission setup that never blocks the flow

`agent-permission-setup` · Notes · design

Operator 2026-10-03, while striking the roadmap's shuttle generic tools-restriction: restricting tools per agent is meaningless today, but lyx needs a correct setup of denied and allowed tools and commands, so nothing gets blocked.
Seen the same day: the global deny list (`git reset --hard`, removed by the operator; `git push --force` still in) stopped `pair-lifecycle` twice at steps lyx itself prescribed.
Related: `no-denied-recovery`, `orch-live-hardening` (permission modes mean what they say).