# Round 1 fixer report — discussion

Every finding's premise was checked against the code and the writer stencil and holds; all seven are fixed in `_lyx/discussion/decision-record.md`.

## Fixed

- **F1** (MEDIUM): D5 now gives the seed call the focus-only schema, so round 1 asks for no `exclude_lenses`, and a round-1 focus file that still carries them has them dropped with a warning naming them.
  The first Acceptance criterion now says the judge calls ask for `exclude_lenses` and the seed calls do not.
  Premise confirmed: `internal/shedadapters/bouncer.go` copies `focusSchemaMarkers(b.cfg.ClusterExcludes)` into both the seed and the judge values.
- **F2** (MEDIUM): D5's union is now restricted to round N's resolved fan, ignoring names that fan lacks, so a mid-generation fan switch keeps only the exclusions that apply.
  The empty-union fallback is now derived without reference to round N-1's run: the current fan minus the union for rounds 2 to N-1, the whole current fan if that is empty too.
  The Exclusion Acceptance criterion covers the new fallback and the fan-switch case.
- **F3** (MEDIUM): D3's Decision now says a lens that emphasizes a rubric item names the item and never paraphrases its rule.
  `edge-cases`, `decision-coverage`, `card-structure` and `verify-tests` are rewritten as pointers naming the rubric items, with the rules themselves removed.
- **F4** (LOW): the test-tier clause is removed from the `acceptance` lens; the tier stays with the plan's `verify-tests` lens through the rubric's verify-coverage item.
  Premise confirmed: the writer stencil says a scenario "names a behavior to cover, not a test to write".
- **F5** (LOW): the `PATTERN-review-round` constraint now says the fixer acts only after the ready marker, which Go writes once it has accepted the consolidated review, in either `fix_start` mode.
  Premise confirmed: `internal/burlerengine/doc.go` has `FixStartParallel` as the default, with the fixer waiting on the ready marker.
- **F6** (LOW): the usage-record Acceptance criterion now names the fan, empty when solo.
- **F7** (NIT): D8's two sentences restating `PATTERN-shuttle-provider-seam` and `PATTERN-durable-vs-ephemeral-state` are deleted; the unreadable-transcript rule, which no constraint carries, stays.

## Disputed

None.

## Deferred

None.
