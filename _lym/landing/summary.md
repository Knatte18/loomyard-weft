# Parts gate refuses an export that exists or breaks a name check

In the `plan-split-bench` replays, Parts-Write fixed export names in `parts.md` that a part's own check refused later, and the part writers cannot change `parts.md`.
Each such defect needed a hand edit before the parts could join.
This change stops those defects at Parts-Write, where they can still be fixed.

## What changes

- **An export is only what the cards create.**
  `Exports` holds a `plan:` handle or a file path the worktree does not hold yet.
  The parts gate and `lym loom validate-parts` refuse, as `export-existing`, any other ref that names a member (it holds `#`) or a path that exists.
  An existing member a part keeps, re-signs or deletes is never an export; the part's `Delivers` names a re-sign or delete, and a later part uses the member by its quarry spelling.
  `planparser.ExportMet` now counts only a Create target or a Rename destination, so `part-export-missing` and the join refusal read "no card creates".
- **Export names face the part's name checks early.**
  `planparser.CheckExportName` runs `handle-malformed` and `fabric-vocabulary-name` on one `plan:` handle, judged as a Create declaration under the plan format's default language.
  The parts check runs it on every part's and shared spot's `plan:` export.
  `CheckFabricVocabularyName` and the new function share one helper, `fabricVocabularyBreak`.
- **Parts-Write checks names against the PATTERN entries.**
  A new step in `loom-template-parts.md` has the agent read the PATTERN entries governing each export's package or kind of name, and their background files, before naming exports.
  The part writer's prompt says every export is new.
- `partsparser.Validate` and `loomshed.NewPartsGate` take the worktree root; the `parts` gate entry requires an absolute `env.WorktreeRoot`.
- `docs/research/plan-split-tune-2026-11-02.md` maps each bench defect to its change and holds the replay table as pending.

## Design decisions

- The task asked the gate to refuse an export naming an existing member whose signature the part keeps.
  A parts file cannot say whether a part keeps or re-signs a member, so the gate refuses every existing member instead, and a re-sign moves to `Delivers`.
  A contract therefore no longer pins a re-sign as a checked export; the joined plan's gate and Plan-Review still see the cards.
- `PATTERN-cli-cobra`'s `Command()` constructor has no gate check; that defect rests on the new Parts-Write step.
- Per the parent's decision recorded after Discussion, this run lands round 1's changes with no replay: stencils are read only from the hub-wide Bolt copy and production takes only `main`.
  The follow-up task `plan-split-replay` runs the replays on the installed `lym` and tunes further within the remaining cap.

## Testing

`lym gate test` tier0 passes over `planparser`, `partsparser`, `loomshed`, `shedrecipe`, `loomcli` and `contracts/stencils`.
New rows cover `CheckExportName`'s checks, `export-existing` for a member glyph, for a path the worktree holds and through `Validate`, a name-check finding on a part export, an export a card only re-signs now failing `part-export-missing`, and the `parts` gate's worktree-root requirement.
