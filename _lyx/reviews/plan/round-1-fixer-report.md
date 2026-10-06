# Round 1 fixer report

All seven findings fixed; nothing deferred.
No git command was run, and `approved:` in `00-overview.md` was not touched.

## Changed

- **F1** — `03-tooling-contracts-prune.md`: dropped `cmd/testtiming#` from `Edit`, rewrote the Intent to say the card edits nothing under `cmd/testtiming` and why, and narrowed `ImpactSummary`.
  `00-overview.md`: Card Index line 3 no longer names `cmd/testtiming`, and `ownership-boundary` now states the rule.
- **F2** — `00-overview.md`: `ownership-boundary` carries the skill-loading carve-out with the record's symbol list, own-name rule and `//testtiming:keep` naming `skills-in-one-turn`; its rationale cites the two parent additions.
  `14-reed-shuttle-prune.md`: the Intent applies the carve-out, naming `TestSkillLoadSequence` and `TestSkillUnknown` in `claudeengine`.
- **F3** — `19-rerun-measure-after.md` (the former card 18): `cmd/testtiming` is excluded from the keep check, the gap fix and the splice, and is still measured read-only; `Uses: cmd/testtiming#` stays.
- **F4** — split the old card 17.
  Card 17 `stencil-state-prune`: `stencil`, `stencilcli`, `stencilstore`, `state`, `cliwire`, `agentname`.
  New card 18 `cli-leaf-prune`: the other fifteen packages, with the `selfreportcli` `failed` entry.
  The rerun card is now 19 (file renamed, heading and `Commit:` renumbered).
  Every package of the old card 17 appears in exactly one of the two.
  Updated the Card Index, both `Applies to` lines, `prune-procedure` step 1 and card 1's two references to the rerun card.
- **F5** — `05-planglyph-prune.md`: the reference now reads `prune-procedure`, step 4.
- **F6** — `00-overview.md` `prune-procedure` step 4: added the cheap-unit-tests rule and the whole-scenario alone-rerun after a merge.
- **F7** — `19-rerun-measure-after.md`: states that every link in the edited report and in `test-suite-prune-rest.md` still resolves (`PATTERN-markdown-link-integrity`).

## Deferred

None.

## Notes

- The new card 18 states that leaf-package import and seam enforcement tests keep their closed sets.
  I added it after drafting a friction-note sentence the answer key did not license and removing it.
- The old card 17 file `17-stencil-misc-prune.md` was renamed to `17-stencil-state-prune.md` so the slug matches its new heading.
