# Plan round 1 — fixer report

Every finding's premise held against the code, and each was fixed as proposed.
`lyx loom validate-plan` reports `ok` after the changes.

## Changes

- F1 (BLOCKING), fixed as proposed: Card 1 drops `internal/webstercli#TestWireHub_LanguageSkills` from its Edit list and drops its webster `verify_tiers` refusal row.
  The existing `EntryWithLeadingSpaceIsRefused` row pins `configLoadError`'s wrapping (`internal/webstercli/wiring.go:151`), and Card 1's `TestLoad_InvalidFailsNamingKey` rows pin the key name.
  Card 4 keeps its own `TestWireHub_LanguageSkills` edit.
- F2 (MEDIUM), fixed as proposed: Card 1's Intent now has `Load` run `stringList` over both keys unconditionally, and says that a file without `verify_tiers` reaches `Load` with the template's empty list through `yamlengine.FillMissing` (`internal/configengine/config.go`).
  The `TestLoad_ResolvesSkills` row with no `verify_tiers` key now says it pins the template fill.
- F3 (MEDIUM), fixed as proposed: Card 1's `TestWire_LanguageSkills` bullet gives the listed row a two-entry `verify_tiers` list and a `wantTiers` column, asserted against the `Language` that `loomCLI.wire` keeps.
  Cards 2 and 3 compare against the row's `wantTiers` column instead of "the file's tiers".
- F4 (LOW), fixed as proposed: Card 4 gives `renderForkWithPack` and `renderRecoveryWithUncommitted` a `verifyTiers []string` parameter that they pass to the renderers.
  `TestRenderPrompts`' two-tier rows call those helpers directly, and `renderFork`, `renderRecovery` and `TestRenderPrompts_ParentDirective` pass nil.
- F5 (NIT), fixed as proposed: Card 1's Callers bullet now rewrites the comment above `loomCLI.wire`'s `languageconfig.Load` call to say the error names the failing key and its own way forward.

## Disputed

None.

## Deferred

None.
