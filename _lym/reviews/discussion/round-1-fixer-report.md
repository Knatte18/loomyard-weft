# Fixer report — discussion round 1

## Changes

All in `_lym/discussion/decision-record.md`, with a Round 1 entry in `_lym/discussion/support-log.md`.

- F1, MEDIUM, fixed as proposed (the review's second route): `board-scope` stays a procedure lens ending in its list.
  D1 says the template header's procedure-lens sentence and `TestConfigTemplate`'s emphasis-line exemption widen to name it.
  Scope In names the header and the exemption, the Constraint names `board-scope` among the procedure lenses, and D1's acceptance line requires the exemption while any other non-procedure lens, `leanness` among them, still fails without the emphasis line.
  The first route was applied first and then reverted after the self-check (see below).
- F2, LOW, fixed as proposed: `speed`'s loom block sets `loom.code_reviewer_fan: ""` in D6, its heading and acceptance line, D7's `doc.go` bullet and Auto-mode assumptions.
  `speed`'s `darn` and `pawl` blocks set `code_max_bounces: 0` and no fan.
  The self-check added `speed` to Goal, Scope In and Open risk 1's operator step.
- F3, LOW, fixed as proposed: D5 admits loom's retired `webster_reviewer_fan` and `webster_fan`, which `MigrateConfig` renames onto `code_reviewer_fan` before the check, under the same bound; the D5 acceptance line covers them.
- F4, LOW, fixed as proposed: D7's second bullet drops loom's `template.yaml` and `internal/configcli/doc.go`.
- F5, LOW, fixed as proposed: Open risk 2 separates a fan one of the six reviewer or fixer fan keys names, which fails every loom config load, from one named only by `discussion_lenses` or `plan_lenses`, which fails when the Discussion-Write or Plan-Write lead resolves it.
- F6, NIT, fixed as proposed: D6 says `solid` and `wide` take the hub's `code_reviewer_fan`, which is the template's value where no layer sets the key.

## Self-check

Every fork returned a report.

- fix-proposal, MEDIUM, the emphasis line on `board-scope` contradicts its "reports only a contradiction" rule: fixed as proposed, switched to the review's exemption route (F1 above).
- fix-proposal, LOW, and fix-consistency, LOW (two findings), Goal and Scope In still name only `light` and `docs`: merged, fixed as proposed.
- fix-consistency, LOW, Open risk 1's operator step leaves out `speed`'s loom block: fixed as proposed.
- fix-claims: no findings.
- fix-way-forward: no findings.

## Disputed

None.

## Deferred

None.
