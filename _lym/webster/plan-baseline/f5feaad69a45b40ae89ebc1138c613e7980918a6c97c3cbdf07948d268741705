# Card 2 — weaver-config

**Create:**
- `internal/weaverengine/doc.go`
- `internal/weaverengine/template.yaml`
- `internal/weaverengine/testmain_test.go`
- `internal/weaverengine#configTemplate`
- `internal/weaverengine#Config`
- `internal/weaverengine#ConfigSpec`
- `internal/weaverengine#ConfigTemplate`
- `internal/weaverengine#ConfigOpenMaps`
- `internal/weaverengine#LoadConfig`
- `internal/weaverengine#ResolveLadder`
- `internal/weaverengine_test#TestLoadConfig`
- `internal/weaverengine_test#TestResolveLadder`

**Edit:**
- `internal/configreg#Modules`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`
- `pattern/PATTERN-config-layers.md`

**Extends:**
- `//docs/overview.md`

**Uses:**
- `internal/configreg/doc.go`
- `internal/configreg#TestRegistration_MatchesDeclarers`
- `internal/websterengine#LoadConfig`

**Intent:** Start `internal/weaverengine` with the `weaver.yaml` config module (D5, D10).

- `template.yaml`, embedded into `configTemplate` with `//go:embed`, holds `ladder` (`sonnet[medium]`, `opus[medium]`, `opus[high]`, a Claude-only default), `fixer_timeout_min: 60` and `verify_gate: webster`, each with a one-line comment.
- `Config` mirrors the keys: `Ladder []string`, `FixerTimeoutMin int`, `VerifyGate string`.
- `ConfigSpec` names the module `weaver`, classifies `verify_gate` `configengine.CategoryGuard` and `ladder` and `fixer_timeout_min` `CategoryNone`.
  `verify_gate` picks the gate key of the plan-level verify Weaver's Done rests on, so a narrower key would hollow that guard (PATTERN-config-layers).
  The bound: the local layer cannot change `verify_gate`; only the tracked worktree file or the template sets it.
  `ConfigOpenMaps` returns `ladder` and the spec carries it as `OpenMaps`, so `lym config weaver --set` carries the list whole, as tapestry's list keys are carried.
- `LoadConfig` loads through `configengine.LoadOrTemplate`, as `websterengine.LoadConfig` does.
  It refuses, marked `configengine.MarkInvalid` and naming the key, a ladder spec `modelspec.Parse` rejects, a non-positive `fixer_timeout_min` and an empty `verify_gate`.
- `ResolveLadder` refuses an empty ladder, and a spec the registry cannot resolve, each naming the key and `lym config weaver` as the way forward.
  Otherwise it returns one `shuttleengine.ModelChoice` per rung, in order, each built by `shuttleengine.NewModelChoice` from the resolved spec.
  The Implementer row calls it when it builds (card 18), so a bad ladder refuses there.
- `configreg.Modules` gains the per-worktree `weaver` row with `Template`, `CheckLoad: checkLoadNamed(weaverengine.LoadConfig, "weaver")`, `OpenMaps` and `Spec`, as `webster`'s row has.
  `TestRegistration_MatchesDeclarers` then sees the new `ConfigTemplate` declarer registered;
  the module's derived `LocalDenied` is `verify_gate`, so `TestModules_EveryTemplateKeyClassified` gains `wantLocalDenied["weaver"] = [verify_gate]`.
- `pattern/PATTERN-config-layers.md`'s one-line set of degrading callers is rewritten with `weaverengine` added.
- `docs/overview.md`'s package tree gains the `internal/weaverengine/` line: Weaver, the cold-start implementer with model escalation.
- `doc.go` opens the package doc: Weaver, the `weaver` implementer of tapestry's implementer row, and its `weaver.yaml` keys.
  Later cards extend it.
- `testmain_test.go` is untagged support: `TestMain` runs `tmuxkit.Main`, as `internal/runplan/testmain_test.go` does.

`TestLoadConfig` is tier0: the template loads with the three defaults; a layer file with an unparseable ladder spec, a zero `fixer_timeout_min` and an empty `verify_gate` each refuse naming the key.
`TestResolveLadder` is tier0: the default ladder resolves to three choices in order; an empty ladder and an unknown alias each refuse naming `lym config weaver`.

**ImpactSummary:** Adds one per-worktree config module to the registry's list; no existing module or key changes.
