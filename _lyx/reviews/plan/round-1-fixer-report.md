# Plan-Review round 1 — fixer report

Every finding was validated against the code and the decision record and found correct; all eight are fixed.
`lyx loom validate-plan` passes the edited plan with informational findings only.

## Fixed

- **F1** (BLOCKING, card 6): wire no longer calls `fabricengine.ReadOrigin`, which I confirmed creates the weft lock directory and seeds the weft exclude through `ensureWeftLockDirAt`.
  Wire now stats `fabricengine.OriginRecordPath(location)` and sets the lazy `pairMergeBase` reader only when the record file exists, so it opens and writes nothing, and an unreadable record or merge base stays a gate error as darn's is.
  `wireDarn` keeps setting the reader unconditionally, so darn is unchanged.
  `TestWire_DarnRecipe` keeps its loom-seed nil assertion over the record-less `hubLocation` fixture and gains a check that a fixture with an origin record file gets the reader with `VerifyWholeDiff` false.
  The card states that no other wire test changes, since the stat answers not-exist in every `hubLocation` fixture.
  `internal/fabricengine#OriginRecordPath` joins Uses.
- **F2** (MEDIUM, card 3): the new `TestLintPatternCmd_OverARepository` and its `internal/loomcli` budget raise are gone.
  The card now extends `TestLintCommentsCmd_OverARepository` (added under Edit), renamed in place to `TestLintVerbs_OverARepository`, so that each step asserts both verbs over one repository.
  `test-budget.yaml` leaves card 3's targets, and the ImpactSummary names the test change.
- **F3** (MEDIUM, card 10): the trim set is now closed: every entry line of 230 bytes or more, each trimmed to at most 200 bytes.
  Each of the 27 existing background files the card appends to is listed under Prosa beside `PATTERN.md`.
  The Intent notes that cards 4, 6 and 7 edit two of them first, and that this card appends below their edits.
- **F4** (MEDIUM, card 10): `pattern/PATTERN-documentation-lifecycle.md` joins Create, and the entry's inline link to `docs/overview.md#documentation-lifecycle` moves into that file.
  The six created files are exactly the trim set's entries that have no background file.
  Measured, the trim set frees more than the 15000-byte target needs, with the new links counted inside each 200-byte line.
- **F5** (LOW, card 1): `TestGofmtFinding` is dropped.
  Its cases are now in-memory subtests at the start of `TestGofmt_ModuleFilesAreFormatted`, and the `cmd/lyx` row rises by one, as D7 says.
  The `new-tests-where-the-record-counted-none` Shared Decision is deleted, since neither card departs from the record any longer.
- **F6** (LOW, card 6): `roundGateFixture.runGate` now carries the re-sign arrow `func (f roundGateFixture) runGate(t *testing.T, mergeBase func() (string, error)) (passed bool, findings string)`.
- **F7** (LOW, card 3): `readOverview` turns CRLF into LF on both paths.
  The scenario's clean-tree step writes the committed overview back with CRLF and asserts that `lint-pattern` still passes.
- **F8** (NIT, card 6): the ImpactSummary now says the call sites are all on this card instead of giving a count, and it names the wire-time stat.

## Disputed

None.

## Deferred

None.
