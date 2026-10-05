# Card 7 — Prefilter the bare weft/warp scan before its regexes

**Edit:**
- `internal/lyxcwd#bareVocabularyTokenIn`

**Intent:**
`bareVocabularyTokenIn` lowercases its input first and returns false at once when the lowercased text contains neither `weft` nor `warp`, before running `vocabExemptPattern` or `weftnamePackagePattern` (D5).
Both regexes only strip spellings that themselves contain `weft` or `warp`, so text lacking both tokens can never hit, and the result is unchanged for every input;
the regexes and the final check run as today on text that does contain one.
`TestEnforcement_FabricVocabulary` (9.6 s in the measured run) spends its time in those regexes over every scanned line, and most lines hold neither token.
This is a pure performance refactor: `TestFabricVocabularyFailures_Fixtures` and `TestEnforcement_FabricVocabulary` already pin the findings, and no test is added.

**ImpactSummary:** one unexported helper in `internal/lyxcwd`'s enforcement test file; its findings are unchanged.
