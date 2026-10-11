# Plan-Review round 1 — fixer report

All four findings were validated against the code and the decision record, and each premise holds.
`lyx loom validate-plan` passes after the fixes, with only the informational `create-new-unit` note on card 5's new integration test file.

## Changes

- **F1 (BLOCKING), fixed as proposed, without the optional accepting row.**
  `TestWriteSeed_ReadSeed_RoundTrip` ("a given start stamp is kept verbatim"), `TestSeed_LegacyParentKey` (parent key, `"bogus"` unknown key) and `TestReadSeed_AbsentDriverDefaultsToGo` in `internal/shedrun/seed_test.go` already cover four of the planned rows through `ReadSeed`, which card 1 routes through `ParseSeed`.
  Card 1's `TestParseSeed` now holds only the unknown-driver and empty-params-key rows, and the card names the existing tests that keep the other four behaviors covered.
  I did not add the happy-path row the fix allowed ("at most one"), because the round-trip test already pins a decoded seed's fields through `ParseSeed`.
  The budget raise of one top-level test is unchanged.
- **F2 (MEDIUM), fixed as proposed.**
  Card 4's `TestShedRecordSpanAndProducers` adds abort rows for a non-RFC 3339 seed `started_at`, a non-RFC 3339 entry `at` and a non-RFC 3339 `last_at` on a folded entry, each naming the slug, the tag's `Name` and the file.
  It also adds a row for a tag that holds `status.json` and no `seed.json`, where the span starts at the earliest transcript line.
  The Intent now states that `seed.json` absent at the tag (`gitrepo.ErrPathNotAtRevision`) means no seed, not an abort.
- **F3 (LOW), fixed as proposed.**
  Card 2's `TestRunTimeByRole` adds a row with two runs whose working intervals overlap in wall-clock time.
  It asserts that "All runs" gives each role the sum of its per-run times and gives idle the per-run sum, that time share divides by the summed spans, and that the explanatory paragraph follows.
- **F4 (LOW), fixed as proposed.**
  Card 1 now places `TreeRunFileRels` in `paths.go` and `ParseSeed` in `seed.go`, and lists `internal/shedrun/paths.go` under `Edit:`.
  It narrows the "every constructor is a `filepath.Join` onto a told `Location`" claim in both `doc.go` and the `paths.go` header to what holds: the on-disk constructors join onto a told `Location`'s `AnchorPath()`, and no constructor resolves a cwd or spawns git.

## Disputed

None.

## Deferred

None.
