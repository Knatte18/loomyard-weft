# Card 12 — Teach the tier guards the `tmux` and `llm` tags

**Edit:**
- `cmd/lyx/tierpurity_test.go`
- `cmd/lyx/tmuxisolation_test.go`

**Intent:**
Before any file moves tier, the two tag-aware guards learn the D6 tags, so each retag card lands against guards that already see its files as tagged.
`knownTierTags` and `isolationTags` become `integration`, `tmux`, `llm` and, until card 13 retires it, `smoke`.
`isolationTagSets` becomes untagged, each tag alone, and `integration,tmux,llm` combined (D8 constraints), plus `smoke` alone until card 13;
exactly one `TestMain` must compile under each set on each platform, which the existing check already asserts per set.
The pairing of `!integration` and `integration` `TestMain` files stays as it is: the `!integration` file is the `TestMain` under the `tmux`-only and `llm`-only sets.
The failure messages that list the tags (`TestTierPurity_UntaggedTestsSpawnNothing`'s "integration or smoke") name the full list.

The existing self-tests extend with rows, not new tests: `TestIsTierTagged_RecognizesKnownTagsList` gains `//go:build tmux`, `//go:build llm` and `//go:build tmux || llm` rows (the disjunction is the shared-helper constraint recorded after Discussion), and `TestTmuxIsolation_FixtureTrees` gains a fixture whose only tagged file is `tmux`-tagged and whose `!integration` `TestMain` must then compile under the `tmux` set, plus one that fails when it does not.

**ImpactSummary:** test-only, confined to the two `cmd/lyx` guard files; the guards accept a superset of today's tags, so no current file changes verdict.
