---
round: 1
ledger:
  - key: refusal-way-forward-background-names-removed-denies
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: comment-names-deleted-unquotedprefixpattern
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: editdirective-test-asserts-static-prose
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: codex-feasibility-lists-removed-sed-i-guardrail
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, with no prior ledger.
The review raised four `consistency` findings, all residue of the removed in-place-edit and redirect-write rows:
a background file (F1), a stale comment (F2), a stencil-prose test assertion (F3) and a research-doc row (F4).
None is gating, and each fix is a local correction.
