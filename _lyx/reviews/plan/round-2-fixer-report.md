# Fixer report — plan, round 2

## Changes

- **F1, fixed as proposed.**
  In card 8 (`08-filter-placeholder.md`, card 7 before the renumbering), `filterArgument` now takes `goos string` as its first parameter.
  The Intent says `renderTierCommand` passes `runtime.GOOS` and that `filterArgument` quotes through `verifyrun.QuoteArg(goos, ...)`.
  A Create line adds `plan:internal/gatecli#TestFilterArgument`, and Tests describes it as a `tier0` table over `goos`: a tier filter holding `%` is refused under `windows` with the exact refusal text, and is quoted under `linux`.
- **F2, fixed as proposed.**
  In card 9 (`09-dotnet-scoper.md`, card 8 before), the `TestGateTest_Scopes` seed-tree paragraph now says each test class carries `[TestFixture]` and one `[Test]` method.
  It also says `CircleTests`' method body or fields name `Circle`, and that `OtherTests` names neither `Circle` nor `Unused`.
- **F3, fixed as proposed.**
  `lyx loom insert-card 6 impactset-diff-reads` created card 6 and renumbered cards 6 to 12 to 7 to 13.
  Card 6 holds the Create lines for `ChangedPaths` and `AncestorBase`, the `internal/impactset/doc.go` Extends line, the export Intent and its own ImpactSummary.
  Card 7 (`07-derive-entry-points.md`) loses those lines, lists both symbols under Uses, and its ImpactSummary now names only `dotnetimpact`.
  The Card Index line for card 7 no longer names the impactset reads.
- **Renumbering follow-up to F3.**
  Every stale card number the insert listed in `prose_mentions` now points at the renumbered card: cards 2 and 9 (11 → 12), card 11 (9 → 10 twice, 6 → 7), card 12 (8 → 9 twice, 9 → 10 twice).
  The moved paragraph now in card 6 says card 11.
- **Self-check, fix-proposal finding.**
  Card 6 gains a Tests sentence: it adds no test, and card 7's `TestDeriveDiff` rows and card 11's `inert_paths` rows test the two wrappers through their callers.
  The wrapped functions already have tests in `internal/impactset/targets_test.go` and `derive_integration_test.go`.

`lyx loom validate-plan` reports `ok: true` after all changes, with only the informational `create-new-unit` findings.

## Self-check

- fix-consistency: no findings.
- fix-claims: no findings.
- fix-proposal, LOW: card 6 has no Tests section for its two exports. Fixed as proposed, with a Tests sentence naming the rows that cover them.
- fix-affected-set: no findings, and no finding was left unchecked.

## Disputed

None.

## Deferred

None.
