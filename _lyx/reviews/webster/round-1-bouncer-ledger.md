---
round: 1
ledger:
  - key: endsessionbyname-leaves-socket-file
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: new-go-comments-hard-wrapped
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised two findings.
F1 maps to `endsessionbyname-leaves-socket-file`: the engine-less `EndSessionByName` kills the tmux server without removing its socket file, against `PATTERN-test-isolation`.
F2 maps to `new-go-comments-hard-wrapped`: new and changed Go comments break mid-sentence, against `PATTERN-comment-line-breaks`.
Both stay open until a fresh review confirms the fixes.
