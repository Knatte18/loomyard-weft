# Round 2 fixer report

## Fixed

- **F1 — keep reasons unbounded.**
  `_lyx/plan/00-overview.md`, `prune-procedure` step 6, now lists Decision 10's four reason classes and states that a candidate with none of them is deleted or folded, never annotated.
  `_lyx/plan/20-rerun-measure-after.md` now checks that each directive's reason is one of those four classes, not only non-empty.
- **F2 — card 18 leftover grouping.**
  Card 18 is split into `18-cli-ide-prune.md` (`clihelp`, `idecli`, `ideengine`, `quarrycli`, `selfreportcli`, `selfreportengine`, `vscode`) and `19-friction-leaf-prune.md` (`batcher`, `buildinfo`, `friction`, `frictionengine`, `modelspec`, `pattern`, `shell`, `tokenvocab`).
  The old `18-cli-leaf-prune.md` is removed, and the rerun card moved from `19-rerun-measure-after.md` to `20-rerun-measure-after.md` with its heading and `Commit:` subject renumbered.
  The Card Index, both `Applies to:` ranges and the "card 19" references in card 1 now read card 20.
  Every in-scope package is still targeted by exactly one card, and the `Commit:` numbers match the card numbers.

## Deferred

Nothing deferred.
