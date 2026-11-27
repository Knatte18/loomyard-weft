```yaml
lens: leanness
outcome: reported
```

Task: leanness lens review of the decision record and support log, looking for content to move or cut.

**MEDIUM: `## Auto-mode assumptions` repeats the Decisions word for word**
Every bullet restates a decision already made:
- the 500 by 600 defaults and their grids (D5)
- `off` as the default mode (D7)
- non-lym windows counted (D6)
- `same` overflowing like `next` (D8)
- no rearrangement on a screen change (D11)

The support log's question ledger already marks each of these as an auto-pick. A plan writer loses nothing if the section shrinks to one line, "every decision is an auto-pick; see the support log ledger", or to one pointer per decision with no restatement.

**MEDIUM: `## Constraints` re-argues decisions instead of stating outside facts**
- "`PATTERN-config-layers`: the machine layer is the only home… (D7)" restates D7's rationale.
- "`PATTERN-child-processes` and `PATTERN-lym-files` bind D3, D9 and D10" repeats D3's own statement about child processes and D9's rationale.
- "`internal/desktophook` stays window-manager-free, and `internal/termwindow` only gains `--separate`…" repeats D3's last line and D2.

Fix: cut all three lines. Constraints should keep only the tool and version facts: the Plasma and KWin versions, the API list, the journal channel and the kscreen JSON fields. Those carry the pins the plan writer needs.

**LOW: the last sentence of D6 is implementation detail**
It spells out how the apply script builds the grid: release the windows, remove the children, split, set `relativeGeometry`, call `manage`. That is plan-level detail, and Notes for the plan writer already points at `set-tiling-grid.sh` as the working reference. Move it into that Notes bullet. The completeness test passes, because the zone order and counting rules stay in D6.

**LOW: D3 spells out exact command lines**
D3 gives the full `qdbus6 … loadScript <file> <plugin>` and `/Scripting/Script<id> run` invocations, and names the temp file under `lymtemp`. The decision itself is the boundary: one package, generated scripts, results read from the journal, and query, plan, apply. The exact invocations belong in Constraints, as the pinned fact of how the operator's scripts call KWin, or in Notes.

**LOW: D13 is bookkeeping, and its acceptance criterion checks existing behaviour**
- Classifying keys is already forced by `TestModules_EveryTemplateKeyClassified`. The criterion "an unclassified key fails the existing template test" asserts nothing the change adds.
- The real content is a category per key and the PATTERN background line. That fits into D5 and D7 (one category clause each) plus the existing `Review:` criterion.
- Cutting D13 also removes a decision id that the acceptance criteria otherwise have to name.

**LOW: the "auto-snap script not found" constraint keeps the search narrative**
The list of places searched is exploration history and already sits in the support log's Turn 1 and Q6. The record needs only the fact: "lym does not depend on an operator auto-snap script; none exists on Neve."

**LOW: Notes repeats Constraints on the contract test**
The second bullet in Notes, that the contract test needs a real Plasma session and is the operator's call, repeats the `PATTERN-test-stand-ins` constraint line. Keep only one of the two.

**LOW: the third bullet of `## Open risks` (`area-host-exits`) is not a question for the parent**
That entry is a draft that depends on this area of the code, so the warning is for whoever implements it. It could be a Constraint line, or a note filed on that entry, rather than an open risk raised to the parent. Keep it, but move it.

**LOW: the Out list in `## Scope` partly duplicates other sections**
The "Rearranging windows when the screen setup changes" bullet restates D11. The VS Code and project bullets duplicate the open risks, though they already point there. Cut the D11 bullet to a pointer.

Further tool calls: none.
