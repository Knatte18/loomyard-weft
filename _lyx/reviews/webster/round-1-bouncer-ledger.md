---
round: 1
ledger:
  - key: go-comments-fixed-column-wrapped
    rounds: [1]
    status: resolved
  - key: gate-go-file-doc-miscounts-exported-types
    rounds: [1]
    status: resolved
  - key: profile-submap-guard-plants-removed-key
    rounds: [1]
    status: resolved
  - key: unknown-gate-name-error-lacks-element-context
    rounds: [1]
    status: resolved
  - key: loomrecipe-fake-gated-doc-stale
    rounds: [1]
    status: resolved
---

Round 1 raised five findings against the gate-list diff.
The most serious was a MEDIUM: new and changed Go comments were wrapped at a fixed column instead of following CONSTRAINTS.md's semantic line breaks.
The other four were LOW or NIT:
a miscounted type list in the `gate.go` file doc, a profile-sub-map guard test that planted the removed `gate_attempts` key, an unknown-gate-name error with no `gates` element context, and a stale `loomrecipe` fake doc.
The Burler round fixed all five in seven commits, and the judge confirmed each fix in the diff, so every entry is resolved.

Two items were looked at and declined rather than recorded.
A few changed lines still open a sentence that continues on an untouched legacy line, but the wrap is legacy.
The loom recipe's YAML comments are wrapped at a fixed column, but CONSTRAINTS.md's convention covers Go comments only.
Neither should be re-raised in a later round unless the convention's scope changes.
