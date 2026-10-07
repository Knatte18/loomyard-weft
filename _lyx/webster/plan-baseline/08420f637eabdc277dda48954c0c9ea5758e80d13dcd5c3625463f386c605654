# Card 10 — Delete the vocabulary exemption list

**Edit:**
- `internal/lyxcwd/enforcement_test.go`
- `internal/lyxcwd/vocabscan_test.go`

**Prosa:**
- `pattern/PATTERN-fabric-vocabulary.md`

**Intent:**
D6: with every exempt spelling renamed or moved by the earlier cards, the exemption mechanism goes, so a non-owner file carries no side-named spelling, owner-set exports included.

- `enforcement_test.go` loses `vocabExemptEntry`, `vocabExempt`, `vocabExemptPattern` and the stripping step in `bareVocabularyTokenIn`; the `weftname` package-name stripping for a file that imports it stays.
  The doc comments of `bareVocabularyToken` and `TestEnforcement_FabricVocabulary` drop their mention of the exempt spellings, and `bareVocabularyTokenIn`'s drops its sentence on both patterns stripping spellings, keeping the `weftname` one.
  `vocabScanAllowlist` keeps its entries and gains none.
- `vocabscan_test.go` drops its "exempt identifier alone passes" row, the only fixture that exercises stripping, and renames its "non-exempt identifier fails" row to name a side-named identifier in a non-owner test file, which is the behaviour that row already pins.
- `pattern/PATTERN-fabric-vocabulary.md` drops its `vocabExempt` bullet and states that a non-owner file carries no side-named spelling, owner-set exports included.

If `TestEnforcement_FabricVocabulary` still reports a non-owner file, that file's spelling is a leftover of an earlier card's grep pass and is reworded here, in code or comment, toward the code/records vocabulary.

Coverage: `TestEnforcement_FabricVocabulary` over the real tree, which fails on any owner-set export still spelled outside the owner set, and the renamed fixture row for a side-named identifier.

**ImpactSummary:** test-only: the vocabulary scan loses its exemption list, and any non-owner file that still spells a side now fails it.
