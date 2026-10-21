---
round: 4
focus:
  - "For every card that adds a func field or a key to a struct, a table or a config module, list each audit that walks that type whole rather than by named field: reflection-based audits such as `envkit.NilSeams` and its three callers (`TestFullEnv_FillsEverySeam`, `TestWire_EverySeamFilled` in `internal/battencli` and in `internal/loomcli`) with `envkit`'s `nilLegal` and each `intentionallyNil` table, plus exact-set pins and enumerating docs. Settle whether every such site is a listed target of some card that precedes or equals the card whose seam breaks it."
  - "For every card whose Intent rewrites a comment, doc paragraph or help text, check that the symbol whose doc or text carries that sentence is a listed member glyph of the card, including `Goto` in `internal/shedengine/goto.go`, `dispositionForHandshake` in `internal/loomcli/bootstrap.go`, the status help texts in `internal/loomcli/cli.go` and `internal/loomcli/doc.go`."
---

Focus targets the method gap, not the individual sites, so the fixer's repair is checked for the whole class.
