# Fixer report — plan, round 3

`lyx loom validate-plan` answers `ok: true` after every change below, its only findings the informational `create-new-unit` notes.

## Changes

- **F1** (BLOCKING), fixed as proposed, with one addition.
  - Card 2: `TestEditOne` and `TestDispatchSet` join **Edit:**, and the Tests paragraph moves their sync rows to the hub case.
    The self-check corrected the `TestEditOne` half: its rows pass `configDirs` with `board` set, as `newHubFixture` builds, since `worktreeDirs` leaves `board` empty and a directory on disk would not change that.
  - Card 15: `newBeginFixture`, `newRecoverFixtureOver`, `newRunFixtureOver` and `webstercli#newVerbsFixture` join **Edit:**, with the Tests sentence on the gate config each seeds.
    The review named `newRecoverFixture`; the self-check found the `RecoverDeps` literal in `newRecoverFixtureOver` (`recoverbatch_test.go:120`), which `newRecoverFixture` and `recoverbatch_git_test.go` both call, so that glyph replaces it.
  - Card 13: `seedDarnHubConfig` is re-signed without `verify`, `TestPerchHardCap` and `TestWire_LanguageSkills` join **Edit:**, and the Tests paragraph states the dropped replacement.
    Addition: `TestWire_DarnRecipe`'s subtest `an empty verify command is refused early with its way forward` goes, since it asserts the `wireDarn` refusal the card already removes.
- **F2** (MEDIUM), fixed as proposed: cards 12 and 10 state the committed `gate.yaml` and the `lyxdirs.DotLyxDirName` exclude in `.git/info/exclude`.
  The self-check added to card 10 that the file is committed in the scratch module's base commit, so no `--base` diff holds it.
- **F3** (MEDIUM), fixed as proposed.
  - Card 10: `RoundGate` gains `GateTiers`, `checkedPublishFailure` and `PublishFailureNote` are re-signed with `tiers`, `TestCheckedPublishFailure_MergeCommit` joins **Edit:**, and the Intent states the mapping's source, the error path, the `none` answer and the `wiring.go` closure's lookup.
    The self-check added that `envkit.FullEnv` and the round fixtures fill `GateTiers`.
  - Card 13: `landingshed.Deps` gains `GateTiers`, filled by `landingDeps` and read by `recordFailure`.
    The self-check added it to the Intent's field list, and stated that it stays out of `nilLegal` and that `LandingDeps` and `newPublishVerifyFixture` fill it.
- **F4** (LOW), fixed as proposed: card 2's drop warning names both steps when the machine layer applies, and the warning subtest asserts both steps for that row and only plain `lyx config gate` for the standalone row.

## Self-check

- fix-consistency, LOW, card 10: `envkit.FullEnv` and the round fixtures were not said to fill `GateTiers`. Fixed as proposed.
- fix-consistency, LOW, card 13: the Intent's `Deps` field list omitted `GateTiers`. Fixed as proposed.
- fix-claims, MEDIUM, card 2: creating a board directory does not give `TestEditOne`'s `worktreeDirs` a board. Fixed as proposed.
- fix-affected-set, MEDIUM, card 15: the `RecoverDeps` literal is in `newRecoverFixtureOver`. Fixed as proposed.
- fix-affected-set, LOW, card 15: more tests use the fixtures than the Tests paragraph names. Disputed (see below).
- fix-proposal, MEDIUM, card 13: nil `GateTiers` and `nilLegal` were left open. Fixed as proposed: the lookup is required and stays out of `nilLegal`.
- fix-proposal, LOW, card 10: the commit that holds the seeded `gate.yaml` was not named. Fixed as proposed.

## Disputed

- fix-affected-set LOW (card 15, the wider set of fixture users): premise false.
  The fork's own evidence says the fixture edit already covers every one of those tests and no glyph is missing.
  The card's sentence says "every test built on these fixtures", which holds for that set as written.

## Deferred

None.
