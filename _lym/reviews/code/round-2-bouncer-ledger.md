---
round: 2
ledger:
    - key: tmux-steer-omits-tier
      rounds:
        - 1
      status: resolved
    - key: ungated-writer-run-form-omits-tier
      rounds:
        - 1
      status: resolved
    - key: tool-learning-rules-miss-code-fixers
      rounds:
        - 1
      status: resolved
    - key: tmux-row-skips-interactive-sessions
      rounds:
        - 1
      status: resolved
    - key: tmux-tier-opening-prose-only
      rounds:
        - 1
      status: resolved
    - key: darntable-one-exception-comment-stale
      rounds:
        - 1
      status: resolved
    - key: tmux-row-hangs-on-autonomous-switch
      rounds:
        - 2
      status: resolved
      class: design
      severity: BLOCKING
      fix: mechanical
    - key: gate-test-lacks-run-agent-tmux-tier-refusal
      rounds:
        - 2
      status: resolved
      class: design
      severity: BLOCKING
      fix: mechanical
---
Round 1 raised six findings; round 2 finds all six resolved.
Two of them, the interactive-session narrowing and the prose-only tmux-tier opening, were settled by parent rulings that the diff did not yet follow.
Round 2 therefore raises them again as two new BLOCKING design findings.
`tmux-row-hangs-on-autonomous-switch` asks for a named spec switch, set by the sampler, in place of `SwitchAutonomous` for the tmux-script row.
`gate-test-lacks-run-agent-tmux-tier-refusal` asks for a `lym gate test` refusal in `internal/gatecli` for a run agent's tmux-tier call that names no package and no filter.
