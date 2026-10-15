# Plan round 1 — fixer report

## Changes

- **F1** (fixed as proposed): card 4's Edit list gains `internal/loomcli#landingDeps` with the re-sign arrow (`languageSkills []string` after `parentName`), `loomCLI.loomPreRun`, `loomCLI.buildLoomShed`, `TestLandingDeps_EveryFieldPopulated`, `TestLandingDeps_ConfigNoticeSeamsOverRealHub`, `markDoneFixtureDir`, `TestWire_Real_PublishRejectedPushNamesRemoteTip` and `TestLandingDeps_VerifyCommandReadsPlanAtCallTime`.
  Card 4's Intent now has `landingDeps` take the parameter and set `landingshed.Deps.LanguageSkills`, the production callers pass `nil`, the four test callers pass `nil`, and `TestLandingDeps_EveryFieldPopulated` passes `[]string{"scribe:golang-testing"}`, asserts the field and drops the "sixteenth field" count.
  Card 4's ImpactSummary names the `landingDeps` parameter.
  Card 5 drops the `landingDeps` line and the five test lines from its Edit list, keeps `loomPreRun` and `buildLoomShed`, and its Intent and ImpactSummary now say `wireLanding` passes `c.languageSkills` where card 4 left `nil`.
- **F2** (fixed as proposed): card 1's `Load` marks invalid and wraps `ErrUnusable` only for content failures (a `LoadOrTemplate` error that is `configengine.ErrInvalid`, plus the loader's own `skills` checks).
  Any other `LoadOrTemplate` error is returned unwrapped and unmarked, so it is classified as transient.
  The `ErrUnusable` sentence now says it marks a content failure.
  `TestLoad_InvalidFailsNamingKey` gains a row with a directory where the board's `language.yaml` belongs, asserting the error is neither `configengine.ErrInvalid` nor `languageconfig.ErrUnusable`.
- **F3** (fixed as proposed): card 1's Intent, after the template paragraph, states what the `skills` hatch lets through and what bounds it, per D7's Bound.
- **F4** (fixed differently in one word): card 8's Intent states what the `language-skills` key lets through and what bounds it.
  The proposed sentence said "both seats of a half"; it says "every seat of a half" instead, since D5 says "every seat of that half" and the support log records that the fix half has one seat.
- **F5** (fixed as proposed): card 1's local-override row creates the anchor's `_lyx/` directory (`configengine.ConfigDir(anchor)`) as well as the local file, and passes both `Board` and `Anchor` in the `Dirs`, so the local layer stacks.

`lyx loom validate-plan` passes with informational findings only.

## Disputed

None.

## Deferred

None.
