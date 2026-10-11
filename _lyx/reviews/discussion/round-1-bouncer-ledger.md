---
round: 1
ledger:
  - key: d7-unresolved-member-glyph-locate-path
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d2-conditions-contradict-reasons
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d2-language-none-handles-outside-closed-set
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: d2-error-path-rule-no-criterion
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: stale-banner-path-note-not-in-scope
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: planindex-wrong-pattern-cited
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised six findings.
The gating one is the D7 pack fragment: it bans quarry and grep for listed members but leaves unresolved member glyphs, whose unit is a package, with no permitted way to be located.
D2 has two further MEDIUM problems: its entry and exit conditions contradict its reasons, and `language: none` plans with handles fit none of its closed reasons.
The LOW findings cover a missing error-path criterion, a stray banner-path note, and a wrong PATTERN citation for `planindex`.
