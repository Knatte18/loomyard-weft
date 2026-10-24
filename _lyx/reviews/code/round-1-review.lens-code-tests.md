```yaml
lens: code-tests
outcome: reported
```

**Lens code-tests. Per behaviour, the test that pins it (or none):**

| Behaviour | Pinning test | Verdict |
|---|---|---|
| One-tag scan refuses any `integration`/`tmux`/`llm` constraint, guard-marked or not (`firstRetiredTierSpelling`, `tierTagFailure`) | `TestTierTagFailure_FixtureSources` (`cmd/lyx/tiertags_test.go` ~146-150): `OldSpellingOnlyTestFunction`, `IntegrationOnlyTestFunction`, `GuardWithIntegrationTerm`, `NegatedRetiredSpellingWithoutTest` | Pinned. The `llm` branch has no row of its own, but all three spellings take the same map lookup. |
| One-tag scan passes a guard file tagged `tier0` alone | Not in the diff. A tier0-only guard row would have to be an existing row, e.g. `UntaggedMainAndHelpers`/guard rows I did not see. | Not verifiable from the diff. |
| LLM guard confines llmkit imports to `tier3` alone | `TestLLMTier_FixtureTrees`: `tier1 file imports llmkit` and `retired llm spelling imports llmkit` both want 1; `tier3` rows want 0. | Pinned. |
| Purity guard: `knownTierTags` and `isTmuxTier` name tier tags only | `TestIsTierTagged_RecognizesKnownTagsList`, both tables: `tier1 || tier2` is false, `tier2 || tier3` is true, and `tier0 || tier1` is false. | Pinned. The old-spelling rows were dropped. After that, `integration`/`tmux`/`llm` count as untagged in purity, and only the one-tag scan refuses them. |
| Isolation guard: `isolationTags` and `isolationTagSets` (6 sets) | `TestTmuxIsolation_FixtureTrees`. I recomputed the counts: "no TestMain at all" = 6, "tier1-only main" = 4, "untagged without TestMain" = 6. All match the new sets. | Pinned. |
| `defaultRedundancyTags` is `tier0,tier1,tier2` | `TestRedundancyTags` row "default is every tier but tier3". | Pinned. |
| `fileTier` reads the tmux tier from `tier2` alone | `TestScanSpawns`, rows `TestTier2Tier`, `TestTier2LinuxTier`, `TestTier0Tier`. | The positive case is pinned. No row shows that an old `tmux` tag is no longer the tmux tier (the acceptance criterion's "alone"). That is a negative the type system does not rule out, but it is optional. |
| `go test -tags tier2/tier3` is refused in the shellguard raw-go row | `TestRows_PatternsDenyAndAllow`: `tier2_tag`, `tier3_tag_with_equals`, `tier3_in_a_tag_list`, `long_tags_flag`; allow rows `tier1_*`. | Pinned. |
| `verify-module-wide` refuses tier2/tier3 | `TestValidate_VerifyModuleWide`: the `tier2`, `tier3` and `tier3 in a list` rows fire, and the `tier1` and `vet tier2` rows are silent. | Pinned. |
| `claudeengine` settings golden follows the shellguard row | I did not read the golden-comparison test. The golden diff follows from the `rows.go` pattern change. | Assumed covered. |
| `impactset` `go list` tags (`derive.go`, `guards.go`) | I found no test that fails if a tier tag is dropped from the list. `FileTier` reading "N of the `tierN` tag" has an existing `tier1` row; its removed row was the `tier1 || integration` case. | Partly pinned (see finding 1). |
| `planglyph` load tags and `tagged.go` under `tier1` | `glyphchain_helper_test.go:207-210` drives `shapes#Tagged`. It fails if `tier1` leaves `BuildFlags`. | Pinned. |
| `allow_tmux.go` constraint `tier2 || tier3` | No dedicated test. The `tmuxisolation` and `tierpurity` guards read the build tags only indirectly. | Not pinned, but a build-constraint line has no behaviour a test could assert. |

**Findings**

1. **LOW, class `consistency`**, `internal/impactset/guards_test.go:~56-60`. The only row removed was the `tier1 || integration` case, and it had a `tier1`-alone twin, so removing it loses nothing. A `FileTier` row for several tier tags reading as the lowest (e.g. `tier2 || tier3` → 2) is not visible in the diff; `FileTier`'s doc comment still states that rule. If no row exists, that rule is now unpinned. I did not confirm either way within my call budget. A row is warranted only if none exists.

2. **NIT, class `consistency`**, `cmd/lyx/tiertags_test.go:146-150`. Three rows all hit the single retired-spelling branch with the same assertion: `OldSpellingOnlyTestFunction`, `IntegrationOnlyTestFunction` and `GuardWithIntegrationTerm`. They differ in constraint shape (`tmux && !windows`, bare `integration`, `tier0 || integration`), so they are defensible. Under test economy they could shrink to one row per distinct shape. No `llm` row exists, which is acceptable.

3. **NIT, class `consistency`**, `cmd/testtiming/spawnscan_test.go:~39`. The fixture is now tagged `tier1`, but the variable `fixtureIntegrationTests` and the file name `integration_test.go` keep the old spelling. This is not an old *tag*, but a reader sees the retired name.

4. **LOW, class `scope`**, leftovers in tests. `cmd/testtiming/splice_test.go:72,101` and `cmd/testtiming/prebuild_integration_test.go:55-63` pass `tags: "integration"` as a tags string. The decision record says prose mentions stay, but these are live test inputs. I did not check whether the code under test treats the value as an opaque non-empty string. If it does, they are harmless. Otherwise they are missed respellings. `cmd/lyx/crosscompile_test.go:9` carries a `-tags integration` comment (prose, allowed). `contracts/stencils/govalues_test.go:102` still bans `"-tags tmux"` and `"-tags llm"` as strings. That is plausibly intentional, but it is an old-spelling literal that the shellguard row no longer names.

5. **LOW, class `consistency`**, `pattern/PATTERN-test-speed.md:19`. It still says the guard file takes `tierN || integration` "until the deployed lyx replaces the installed guard scan, … the hub's tier commands carry both spellings". This is the old transitional state. The handler and the pattern-fit lens own it; I list it only because it is the doc counterpart of the test change.

**Tool calls:**
1. Read the persisted test diff `benlx2wq3.txt`; my context held only the first 2 KB of it.
2. Ran one grep over the `impactset`, `planglyph` and `testtiming` tests for remaining old-spelling or multi-tier rows. The manifest did not show these tests' contents.
