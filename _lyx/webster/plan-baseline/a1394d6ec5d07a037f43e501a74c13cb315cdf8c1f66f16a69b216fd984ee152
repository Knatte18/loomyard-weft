# Card 16 — tier-purity-tier-names

**Edit:**
- `cmd/lyx#knownTierTags`
- `cmd/lyx#isTierTagged`
- `cmd/lyx#isTmuxTier`
- `cmd/lyx#TestTierPurity_UntaggedTestsSpawnNothing`
- `cmd/lyx#TestIsTierTagged_RecognizesKnownTagsList`
- `cmd/lyx#canCompileWithoutLLM`
- `cmd/lyx#llmTierFailures`
- `cmd/lyx#TestLLMTier_OnlyLLMFilesReachAnLLM`
- `cmd/lyx#TestLLMTier_FixtureTrees`
- `cmd/lyx#isolationTags`
- `cmd/lyx#isolationTagSets`
- `cmd/lyx#TestTmuxIsolation_FixtureTrees`

**Intent:**
Restate `cmd/lyx/tierpurity_test.go` and `cmd/lyx/llmtier_test.go` with the tier names (D5), before any file is retagged, so a `tier0` file is held to the untagged rules.

The purity guard's tagged set becomes `tier1`, `tier2` and `tier3` beside `integration`, `tmux` and `llm` until D13's follow-up drops the old spellings, and `isTierTagged` reads a file as tagged when its constraint can hold only with one of those tags: a file tagged `tier0`, `tier0 || integration` (D13) or nothing at all counts as `tier0` and is scanned for banned spawns and long sleeps as an untagged file is today.
`isTmuxTier` treats `tier2` and `tier3` as it treats `tmux` and `llm`.
The LLM guard accepts `tier3` beside `llm` as the only tier that may reach an LLM: `//go:build tier3` and `//go:build tier3 && linux` pass, `tier2 || tier3` fails.
`TestIsTierTagged_RecognizesKnownTagsList` gains rows for `tier0` (untagged), `tier0 || integration` (untagged), `tier1`, `tier2`, `tier3` and `tier2 && !windows`; `TestLLMTier_FixtureTrees` gains a `tier3` fixture and a `tier2 || tier3` one.
The tmux isolation guard (`cmd/lyx/tmuxisolation_test.go`) compiles each package's test files under the tier tags beside the old ones: `isolationTags` adds `tier0` through `tier3`, and `isolationTagSets` adds each tier tag alone and all of them together, so a retagged package stays in its view; `TestTmuxIsolation_FixtureTrees` gains a fixture package whose test files are tagged `tier0` and `tier2`, which the guard must still check.
The guards' failure messages name the tier tags.

**ImpactSummary:** Three repository-scanning guards learn the tier tags and count `tier0` as untagged; no production code changes.
