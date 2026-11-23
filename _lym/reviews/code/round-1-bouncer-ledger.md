---
round: 1
ledger:
    - key: tmux-steer-omits-tier
      rounds:
        - 1
      status: resolved
      class: design
      severity: BLOCKING
      fix: mechanical
    - key: ungated-writer-run-form-omits-tier
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: tool-learning-rules-miss-code-fixers
      rounds:
        - 1
      status: open
      class: scope
      severity: MEDIUM
      fix: mechanical
    - key: tmux-row-skips-interactive-sessions
      rounds:
        - 1
      status: open
      class: decision
      severity: MEDIUM
      fix: mechanical
    - key: tmux-tier-opening-prose-only
      rounds:
        - 1
      status: open
      class: decision
      severity: LOW
      fix: mechanical
    - key: darntable-one-exception-comment-stale
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
---
Round 1 raised six findings.
The tmux-script steer and the ungated writer stencil both give a `lym gate test` way forward without `--tier`, so a tmux-tier test would be skipped under tier0.
The tool-learning and no-background rules reach only the darn writer fragments and the cold gate fixer.
Two findings on narrowing choices (autonomous-only row, prose-only tmux-tier opening) are deferred to the parent.
One stale doc comment in `internal/darnengine/spec.go` remains.
