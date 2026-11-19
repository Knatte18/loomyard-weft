```yaml
lens: repo-invariants
outcome: reported
```

**Repo-invariants lens: what each card checks, and whether it holds or breaks**

- Card 1: the gatecli and gateslot docs are extended (held). No reflective seam test covers `testRequest` (held). The new tests are tier0 (held). Card order holds.
- Card 2: `TestConfig_TierUse` and `templateDefaults` are listed (held). Tier0 (held). Order holds.
- Card 3: the new `internal/hardenshed` test package has no `TestMain` calling `tmuxkit.Main` (BROKEN, F2). `PATTERN-config-layers`' strict-caller list leaves out the new strict loader (BROKEN, F6). `configreg` module tests are listed (held). Order holds.
- Card 4: no test cross-checks `LoomBuildEngines` against `registry` before card 11, so card order holds. The shedbuild doc is listed (held).
- Card 5: `envkit` `nilLegal` and `FullEnv` are listed for `PreflightHarden` (held). No reflective test walks `preflightshed.Checks` (held). It uses cards 1 and 2 (order holds).
- Card 6: tier0 (held). It uses card 1's parser (order holds).
- Card 7: `SandboxDeps` spells the fabric side tokens in a non-owner package (BROKEN, F1). Tier tags hold. The lymdirs and fabricengine docs are extended (held).
- Card 8: `internal/runproc/doc.go` is falsified (BROKEN, F4). The `PATTERN.md` and background edits are listed (held). `TestStopServer` is tier2 (held).
- Card 9: docs are extended and tiers hold. It uses card 8 (order holds).
- Card 10: no reflective test walks `BouncerConfig` (held). The shedadapters doc is extended (held).
- Card 11: `envkit` `Harden` is listed (held). The coverage guard entry is added here and dropped in card 13 (held).
- Card 12: `.gitattributes` and `entries` are listed (held).
- Card 13: the closed lists and their tests are listed (held). The doc extends hold.
- Card 14: `TestWire_HardenerRecipe` is tier1 and the other new tests are tier0 (held). The loomcli doc is extended (held).
- Card 15: `Landing.Reap` is in `nilLegal`, and `TestLandingDeps_EveryFieldPopulated` (`internal/loomcli/landingdeps_test.go:138`, a reflective field walk) is listed (held). It edits card 14's test (order holds).
- Card 16: the new `internal/hardencli` test package has no `tmuxkit.Main` `TestMain` (BROKEN, F2). `PATTERN-cli-cobra`'s package-naming deviation list is falsified (BROKEN, F3). `TestSandboxVerbs` is tier1 and uses no tmuxkit spawn (held).
- Card 17: `fakeSubstrates` and `TestTeardown_CallSequence` are listed, and no reflective walk covers `Teardown` (held). It uses card 7's lymdirs name (order holds).
- Card 18: comes before card 19 (order holds).
- Card 19: `internal/mergeresolve/seam_enforcement_test.go`'s comment still names `treadleengine` (BROKEN, F5; the affected-set lens may also report it). `PATTERN-cwd-resolution.md` is listed (held). No golden file outside `agentconfig/template.yaml` names treadle (held).
- Card 20: `fabricvocab.docRoots` and `PATTERN-fabric-boundary.md` are listed (held). It edits card 3's doc (order holds).

**Findings**

- **F1. BLOCKING, design, card 7 (`plan:internal/hardenshed#SandboxDeps`).** `SandboxDeps.Clone func(warp, weft, container string)`, and `Seed` returning `(warp, weft string, err error)`, are wired into `hardenshed`. That puts bare side tokens into a non-owner package's code. `PATTERN-fabric-boundary`, Fabric vocabulary, says "a non-owner file carries no side-named spelling", and `TestEnforcement_FabricVocabulary` scans non-comment code (`internal/lymcwd/enforcement_test.go:23-30` strips only comments). The card's prose ("bare warp ... empty bare weft") invites the same spelling in `hardenshed/doc.go` too. Fix: name the parameters and fields by role, for example `codeBare` and `recordsBare`. `fabricengine.SeedLocalBares` itself sits in the owner set.
- **F2. MEDIUM, consistency, cards 3 and 16.** Each card makes the first test of a new package (`internal/hardenshed`, `internal/hardencli`), but neither lists a `testmain_test.go` whose `TestMain` calls `tmuxkit.Main`. `PATTERN-test-isolation` requires one in every test package, and `cmd/lym/tmuxisolation_test.go:2,30` enforces it with an empty allowlist. Fix: each card creates `internal/&lt;pkg&gt;/testmain_test.go` calling `tmuxkit.Main`.
- **F3. BLOCKING, design, card 16 (`internal/hardencli`).** `PATTERN-cli-cobra`, Package naming, says "`&lt;module&gt;cli` imports `&lt;module&gt;engine`" and lists each deviation. `hardencli` uses `hardenshed` and `loomcli` and has no `hardenengine`. No card adds the deviation to `pattern/PATTERN-cli-cobra.md`, and the overview's `## PATTERN changes` does not declare it. Fix: declare and make the deviation line, or ask the parent to record it.
- **F4. LOW, consistency, card 8.** After the card, `internal/runproc/doc.go:64` ("EndProcess runs the same sequence on one unregistered pid, for the orphans `lym tender reap` ends") is false. The Hardener sandbox reaper becomes a second user, and card 8 updates only `PATTERN.md` and the background file. Fix: card 8 extends `internal/runproc/doc.go`.
- **F5. LOW, consistency, card 19.** `internal/mergeresolve/seam_enforcement_test.go:26`, the doc comment of `mergeresolveAllowedImports` (package `mergeresolve`), cites "the same call treadleengine's seam allowlist already makes". After the deletion it names a package that no longer exists. Card 19 rewords only shedengine's matching header. Fix: add `internal/mergeresolve#mergeresolveAllowedImports` to card 19's Edit list.
- **F6. LOW, consistency, card 3.** `PATTERN-config-layers`, Strictness, lists the strict callers `{fabricengine, boardengine, loomengine, landingshed, gateslot, configcli, configreg}`. Card 3's `hardenshed.LoadConfig` "loads strictly", and no card adds `hardenshed` to that list. No test enforces the list; I found none, and `darnengine` is already missing from it. Fix: add it to the list as a declared PATTERN change, or drop the finding as pre-existing staleness.

**Tool calls**

1. `cmd/lym/tmuxisolation_test.go` and a grep of the test-speed tier rule, plus shedcli, battencli, runproc and configreg doc lists, treadle in non-Go files, and strict-caller enforcement: the manifest did not cover these enforcement sites or package docs.
2. A grep for reflective seam tests over `BouncerConfig`, `Checks`, `Teardown` and `Deps`, the envkit pointer walk, CLI and engine naming enforcement, config module lists, and the gate-slot lease rule: needed to check the seam and doc invariants.
3. A grep for module and recipe enumerations in the configcli tests and `docs/overview.md`, plus `battencli/wire.go:600-620`: to confirm no other closed list misses the new unit or recipe.
4. A grep of `internal/lymcwd/enforcement_test.go` and the `fabricvocab` predicates: to confirm the vocabulary scan matches identifiers in code, not only comments.
