# Agents run with a deliberate tool and permission setup that never blocks the flow

`agent-permission-setup` · Note · enhancement, shuttle

Operator 2026-10-03, while striking the roadmap's shuttle generic tools-restriction: restricting tools per agent is meaningless today, but lyx needs a correct setup of denied and allowed tools and commands, so nothing gets blocked.
Seen the same day: the global deny list (`git reset --hard`, removed by the operator; `git push --force` still in) stopped `pair-lifecycle` twice at steps lyx itself prescribed.
Related: `no-denied-recovery`, `orch-live-hardening` (permission modes mean what they say).

Seen 2026-10-04 in `board-model` (#361): Webster Master ran `python3 - …` with a misplaced heredoc, the shell hung on stdin and blocked the verify gate for 75 minutes.
Operator: lyx agents never use Python. Nothing denies it today: agents run in bypass, and shuttle's PreToolUse hook (`internal/shuttleengine/claudeengine/settings.go`) guards only other cases. Want: the same hook denies `python`/`python3` in Bash for every lyx-spawned agent, with a reason pointing to Edit/Write and grep/awk.
