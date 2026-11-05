# Plan round 1 — fixer report

## Changes

- F1 (LOW, consistency), fixed as proposed.
  The premise holds: `docs/overview.md:319` holds the `internal/reportconfig/` tree line and `:480` lists `internal/reportconfig` in the report module line, and no card's Intent named either.
  Card 08's Intent (`_lym/plan/08-report-cost.md`) gains a sentence after the overview sentence: the package tree gains an `internal/reportconfig/` line describing the hub-wide `report.yaml` config, and the report module line's package list names `internal/reportconfig` beside `internal/reportcli` and `internal/reportengine`.
  The Edit list already held `//docs/overview.md`, so no target changed.
- Gate findings `create-already-exists` on cards 07–12 (blocking), fixed.
  Batch 7's commits (`5ec85b9c` through `d7f0465e`) built every one of these cards' Create targets before the batch was recorded failed, and the plan gate skips tree-dependent checks only for cards Webster's run record holds done.
  Each card's Create targets move under its Edit group, as plain glyphs with no `plan:` prefix and no declaration arrow: card 07's `internal/reportconfig` files and members, card 08's cost types, functions, test and `costCommand`, card 09's raddle path accessors and `RaddleDirName`, card 10's spec file and `RaddleProposalSpec`, card 11's `SectionBullets`, `PartSplice`, `SplicePart` and their tests, and card 12's `internal/raddleparser` files and members.
  Cards 10 and 11 had no Edit group and gain one; cards 07–12 have no Create group left.
  The two `testmain_test.go` file targets (cards 07 and 12) become the member glyphs `internal/reportconfig_test#TestMain` and `internal/raddleparser#TestMain`, since `edit-test-file` refuses a test file under Edit.
  Every `Uses` reference to these now-existing symbols drops its `plan:` prefix: card 08 (`reportconfig#LoadConfig`), card 12 (`discussionparser#SectionBullets`), card 19 (`raddleparser` `Parse`, `Check`, `RecheckBoard`, `BoardContext`), card 20 (`raddleparser#Hash` and four `loomengine` raddle paths), card 22 (`raddleparser` `Parse`, `Hash`, `discussionparser#SplicePart`) and card 26 (`raddleparser#Parse`, `reportengine#usageFiles`, `reportcli#costCommand`).
  `lym loom validate-plan --overlay` with `{}` now answers `ok: true`, with only the two informational `create-new-unit` findings on cards 21 and 23.

## Self-check

The fork pass ran once, over the F1 change, before the gate findings arrived; this session spawns no second pass, so the gate repair was checked by the validator alone.

- fix-consistency: no findings.
- fix-claims: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings. Its out-of-lens note, that card 08's package-list sentence and card 27's "the `report` config module" sentence touch the same overview line, is no finding: fix-consistency and fix-proposal both judged the two sentences to describe different parts of that line without contradiction.

## Disputed

None.

## Deferred

None.
