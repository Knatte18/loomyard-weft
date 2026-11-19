# Remove lym verbs and features nobody uses

`remove-unused-verbs` · Task · enhancement, cli · draft

## Source

`docs/research/cleanup/unused-features-2026-11-18/report.md` (`702fae2d`) lists 34 of 250 verbs that no Claude transcript ran from 2026-10-28 to 2026-11-18,
and the config keys no layer of this hub overrides.

## Before release

The operator picks which candidates go; this body then lists them, and only those are removed.
Not yet decided (operator, 2026-11-18: a follow-up, not now).

Strongest candidates, named by nothing outside their own package:
`board rerender`, `ide menu`, `board retire-legacy`, `creel hold`, `creel release`, `webster await-batch`, `loom restore-renumber`,
`orch withdraw`, `slack away`, `stencil promote`, `report raddle`, `fabric diff`, `config menu`.
`claude account` is new (landed 2026-11-18) and is not a candidate.
`creel forward` stays: the operator expects to need it, as it was designed for (operator, 2026-11-19).

Weaker candidates, named by a stencil, recipe or another package: see the report's second table; each caller needs a look before removal.

## Scope once picked

Remove each chosen verb, its engine code that nothing else reaches, its tests, and every doc, stencil and refusal text that names it.
