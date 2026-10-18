# Card 3 — orchestrator-md-removal

**Delete:**
- `//ORCHESTRATOR.md`

**Edit:**
- `//CLAUDE.md`
- `//contracts/stencils/orch/orch-template-role.md`

**Intent:** Delete `ORCHESTRATOR.md` and reword the remaining mentions of it to D5's placement rule, now that card 1 renders the two rules files and card 2 ships `lyx orch rules`.
Its two rules are not lost: D4's operator step moves them into the hub-wide file after the deploy, and card 4 files that step on the board.

`CLAUDE.md`'s "Notes go in git" line "A rule or workaround only the orch needs goes in `ORCHESTRATOR.md`, never in `CLAUDE.md`." becomes: a rule or workaround only the orch needs goes in the orch rules, never in `CLAUDE.md`: the hub-wide file on Bolt, `_board/_lyx/orch/rules.md`, written through `lyx orch rules`, when every orch of the hub needs it, and the orch thread's own `_lyx/orch/rules.md`, committed with `lyx fabric commit`, when only that thread does; one sentence per line.

The role stencil's two placement sentences say the same:
- In `## Where a finding goes`, the workaround clause becomes: the workaround is written down with the issue number in the hub-wide rules file, `_board/_lyx/orch/rules.md`, through `lyx orch rules`, when only orchs apply it, and in `CLAUDE.md` only when every agent needs it, and removed with the fix.
- In `## Scratch and notes`, "durable orch-only rules in the repository's `ORCHESTRATOR.md`" becomes: durable orch-only rules in the hub-wide rules file through `lyx orch rules` when every orch of the hub needs them, and in this orch's own `_lyx/orch/rules.md` through `lyx fabric commit` when only this thread does.
- One added sentence in `## Scratch and notes` says `lyx orch rules` replaces the whole hub-wide file, so a draft in `.scratch/` starts as a copy of the current `_board/_lyx/orch/rules.md`.

No tracked source, stencil or doc names `ORCHESTRATOR.md` after this card outside Scope's exclusions (`crucible/`, `docs/research/` and generated `_lyx/` content); the card's Verify greps the whole tree but those for it, and passes only on grep's no-match exit, so a grep error fails it too.

**Verify:** grep -rIln --exclude-dir=.git --exclude-dir=crucible --exclude-dir=research --exclude-dir=_lyx --exclude-dir=.lyx --exclude-dir=.scratch "ORCHESTRATOR.md" . ; test $? -eq 1

**ImpactSummary:** Removes a file only card 1's now-retired path read and rewords two prose files; no code reads either, and the stencil's markers stay as card 1 left them.
