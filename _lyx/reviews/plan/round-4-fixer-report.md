# Round 4 fixer report

Verdict was APPROVED; all four findings were fixed anyway.

## Changed

- **F1** (cards 14 and 16 mixed a slow fixture-merge package with unrelated unit packages):
  - Card 14 `reed-shuttle-prune` is now `14-reed-prune` (`internal/reedcli`, `internal/reedengine/render`), and the new `15-shuttle-prune` holds `internal/shuttlecli` and `internal/shuttleengine/claudeengine` with the skill-loading carve-out.
  - Card 16 `fabric-geometry-prune` is now `17-fabric-hub-prune` (`internal/fabriccli`, `internal/hubforge`, `internal/pairteardown`), and the new `18-geometry-prune` holds `internal/hubgeom`, `internal/standalonegeom`, `internal/standalonestate` (with the `PATTERN-leaf-packages` rule), `internal/lyxcwd` and `internal/weftname`.
  - Later cards were renumbered: webster-orch 16, git-fs 19, stencil-state 20, cli-ide 21, friction-leaf 22, rerun-measure-after 23.
  - Updated to match: file names, card headings, `Commit:` subjects, the Card Index, both `Applies to:` ranges (`2–23`, and `2–22` plus card 23), and card 1's references to the rerun card.
  - Checked afterwards: heading, file name, index line and commit number agree for every card, no stale reference to the old card 20 or 21 remains, and the set of `Edit:` packages is identical to before the split.
- **F2** (failed-entry verdicts produced twice with no hand-off): removed the failed-entry run from card 1; `prune-procedure` step 2 and the package cards own Decision 8.
- **F3** (card 13 label omitted `statuscommit`): the Card Index line, the Intent's first sentence and the ImpactSummary of card 13 now name the status-commit package.
- **F4** (wrong decision cited for `cmd/testtiming`): card 23 now cites the parent's `cmd/testtiming` addition to the decision record.

`approved:` in `00-overview.md` was not touched.

## Deferred

Nothing.

## Process note

I ran one `sed` as a no-op in a pipeline during review (it changed nothing), against the no-`sed` rule; no file was edited with it.
