---
round: 1
ledger:
  - key: empty-role-matches-map-entry
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: zero-config-sentinel-in-rejected-test
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: ttl-rows-omit-key-in-expected-error
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: ttl-doc-comment-chained-so-clauses
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 is the first round, so there is no prior ledger.
The review raised four findings, all LOW or NIT.
`empty-role-matches-map-entry` (F1) is the only `design` finding: `resolvePromptCacheTTL` looks up an empty role in the map.
The other three concern test and comment consistency: F2 is the zero-Config sentinel, F3 the TTL rows that omit the key, F4 the doc sentence with two "so" clauses.
