---
round: 2
ledger:
  - key: hard-wrapped-go-comments
    rounds: [1, 2]
    status: open
  - key: stale-config-menu-entry-in-overview
    rounds: [1]
    status: resolved
  - key: stringnode-missing-doc-comment
    rounds: [1]
    status: resolved
  - key: independent-clauses-on-one-line
    rounds: [2]
    status: open
---

Round 1 raised three `consistency` findings.
Round 2's review confirms the `config` bullet in `docs/overview.md` agrees with the CLI, and that every helper in `set.go` and `reconcile.go` carries a doc comment, so `stale-config-menu-entry-in-overview` and `stringnode-missing-doc-comment` are resolved.

`hard-wrapped-go-comments` stays open: F1 finds the new `findValueNode` doc in `internal/yamlengine/reconcile.go` wrapped at a fixed column.

`independent-clauses-on-one-line` is new in round 2.
F1 (the `Reconcile` doc), F2, F3, F4, F5 and F6 each find an independent clause, joined by a semicolon or ", and", on one line in yamlengine, configengine, configcli and boardengine comments and `doc.go` files.
All are BLOCKING and cite `PATTERN-comment-line-breaks`.
