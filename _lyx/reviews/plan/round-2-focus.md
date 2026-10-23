---
round: 2
focus:
  - "Cards 4 and 5 (`_lyx/plan/04-plan-gate-pattern-checks.md`, `05-plan-gate-pattern-base-wiring.md`): does `pattern-removes-prescribed` ask for an edit that `done-card-edited` refuses when the fencing card is recorded done, and does every PATTERN-checks way forward name a card that can still be edited?"
  - "Card 12 and card 14 (`12-settle-round-start.md`, `14-settled-guard-amendment.md`): does every refusal the settled guard can return name a way forward the parent can act on, including a `round_started_at` that does not parse?"
  - "Card 18 and card 19 (`18-recipe-amendment-target.md`, `19-arbiter-amendment-target.md`): is `loomrecipe.AmendmentTarget`'s signature consistent with the package's other per-recipe readers, and does card 19's call match it?"
  - "Card 27 (`27-publish-failure-note-trace.md`): does any card read a glyph string with a local regex or parser instead of `glyph.Parse` and `Glyph.String`, per `PATTERN-glyph-conversion-chokepoint`?"
---

The focus lists the places where a fix changes a signature, a behavior or a refusal's text, so the new text gets a fresh read.
