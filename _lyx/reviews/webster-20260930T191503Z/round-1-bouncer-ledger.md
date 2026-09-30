---
round: 1
ledger:
  - key: refusal-spec-missing-rows
    rounds: [1]
    status: resolved
  - key: refusal-spec-cells-disagree-with-messages
    rounds: [1]
    status: resolved
  - key: recover-batch-way-forward-unparseable-arg
    rounds: [1]
    status: resolved
  - key: rebaseline-fabric-sync-no-way-forward
    rounds: [1]
    status: resolved
  - key: master-ladder-missing-recover-batch-batch-failed-rung
    rounds: [1]
    status: resolved
  - key: goto-refusals-missing-way-forward
    rounds: [1]
    status: resolved
  - key: fingerprint-mismatch-missing-way-forward-marker
    rounds: [1]
    status: resolved
  - key: websterengine-doc-inaccuracies
    rounds: [1]
    status: resolved
  - key: stale-goto-era-docs-and-comments
    rounds: [1]
    status: resolved
  - key: rebaseline-refusal-blank-start-sha
    rounds: [1]
    status: resolved
  - key: go-comment-mid-sentence-hard-wraps
    rounds: [1]
    status: resolved
  - key: fabric-sync-tests-do-not-take-way-forward
    rounds: [1]
    status: resolved
  - key: batten-goto-example-wrong-row
    rounds: [1]
    status: resolved
  - key: duplicate-containsString-and-missing-rebaseline-header
    rounds: [1]
    status: resolved
  - key: refusal-spec-loom-intro-except-that-break
    rounds: [1]
    status: open
---

Round 1: the Burler raised 14 findings, F1–F14.
Five were MEDIUM: missing spec rows, spec cells that disagreed with the messages, recover-batch naming an argument its own parse rejects, rebaseline's sync refusal with no way forward, and a missing Master rung.
Seven were LOW and two were NIT.
The fixer landed one commit per finding (F1 and F2 shared one, and F11 took two).
The judge re-verified each fix in the code, and the full verify suite passes at HEAD `224d017f2`.
All 14 are resolved.

The one open entry is a non-blocking NIT the F1 fix introduced.
The loom-section intro of `contracts/specs/refusal-spec.md` breaks a line before ", except that…", which is not an independent-clause boundary under CLAUDE.md's markdown rule.
It did not stop approval.
