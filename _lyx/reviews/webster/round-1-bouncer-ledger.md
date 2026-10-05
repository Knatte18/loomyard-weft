---
round: 1
ledger:
  - key: hard-wrapped-go-comments
    rounds: [1]
    status: open
  - key: stale-config-menu-entry-in-overview
    rounds: [1]
    status: open
  - key: stringnode-missing-doc-comment
    rounds: [1]
    status: open
---

Round 1 raised three `consistency` findings.
F1 maps to `hard-wrapped-go-comments`: new Go comments in yamlengine, boardengine, configreg, configcli and selfreportcli break sentences at a column (BLOCKING, cites `PATTERN-comment-line-breaks`).
F2 maps to `stale-config-menu-entry-in-overview`: the `config` bullet in `docs/overview.md` still names the removed menu (MEDIUM).
F3 maps to `stringnode-missing-doc-comment`: `stringNode` in `internal/yamlengine/set.go` lacks a doc comment (NIT).
All three stay open until a fresh review sees the fixed text.
