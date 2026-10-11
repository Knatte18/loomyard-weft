---
round: 2
ledger:
  - key: d7-unresolved-member-glyph-locate-path
    rounds: [1]
    status: resolved
  - key: d2-conditions-contradict-reasons
    rounds: [1]
    status: resolved
  - key: d2-language-none-handles-outside-closed-set
    rounds: [1]
    status: resolved
  - key: d2-error-path-rule-no-criterion
    rounds: [1]
    status: resolved
  - key: stale-banner-path-note-not-in-scope
    rounds: [1]
    status: resolved
  - key: planindex-wrong-pattern-cited
    rounds: [1]
    status: resolved
  - key: d7-plan-handle-locate-instruction
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: d2-owned-handles-source-unstated
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: d2-index-lookup-error-omitted
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: support-log-banner-disposition-stale
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised six findings, and round 2 does not re-raise any of them; all six are resolved.
Round 2 raised four new ones.
The two MEDIUM `design` findings concern D7's pack fragment, which gives unresolved `plan:` handles a locate instruction that cannot apply, and D2, which names no source for a card's owned handles on the paths that skip `BindHandles`.
The two LOW `consistency` findings cover D2's closed no-line list omitting the `Index` lookup error, and a support-log line that still claims the banner-path note was fixed.
