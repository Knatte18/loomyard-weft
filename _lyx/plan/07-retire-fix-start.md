# Card 7 — retire-fix-start

**Edit:**
- `internal/burlerengine#RunOpts`
- `internal/burlerengine#Engine.Run`
- `internal/burlerengine/engine_test.go`
- `internal/burlerengine/doc.go`
- `internal/loomengine/config.go`
- `internal/loomengine/template.yaml`
- `internal/loomengine/config_test.go`
- `internal/loomengine#ReviewSettings`
- `internal/loomengine#ResolveReview`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#burlerRoundEntry`
- `internal/shedrecipe/entries_burler_test.go`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli/wiring_test.go`
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`

**Intent:**
Retire the run-wide `loom.yaml` key `fix_start` and its `after-review` value (decision D5) before the round moves onto the seats engine, whose chair-first start order leaves no place for it.

`RunOpts` loses its `FixStart` field, and every use of the `FixStart` type and its two constants goes; card 11 deletes their declarations once card 8 has rewritten the round.
`Engine.Run` loses the unknown-value refusal and the after-review branch, so it always starts the reviewer and then the fixer and joins them, which is today's default `parallel` order.
loom's `Config` loses `FixStart`, `LoadConfig` loses the `fix_start` value check, `config.go`'s header comment drops the `fix_start` clause, and `template.yaml` drops the key.
`ReviewSettings` loses `FixStart` and `ResolveReview` stops filling it; `shedrecipe.Env` loses `FixStart` and `burlerRoundEntry` stops setting it on the round's `RunOpts`; `loomCLI.wire` stops passing it.

A hub `loom.yaml` that still carries `fix_start` loads, since a key outside the template is ignored, and `lyx config reconcile --apply` reports and removes it as a removed key through the template (`PATTERN-config-strictness`, `PATTERN-binary-change-reconcile`); nothing edits a hub file by hand.
The `config: unknown fix_start` row leaves `contracts/specs/refusal-spec.md`, since the refusal is gone.

Tests:
- `loomengine/config_test.go`: the template-defaults `Config` drops `FixStart`; the `fix_start after-review` row of `TestLoadConfig_Loads` becomes a row loading the template plus `fix_start: after-review` as a retired key, which loads to the template defaults, beside the retired `selfreport` row; the `unknown fix_start` refusal row is removed.
- `burlerengine/engine_test.go`: `TestEngine_Run_RoundFailureRules` loses its `fixStart` field and the unknown-fix-start and after-review rows, and the after-review marker assertion.
- `shedrecipe/entries_burler_test.go`: `TestBurlerRoundEntry_EnvReviewFallback` drops the `FixStart` setting and assertion.
- `loomcli/wiring_test.go`: `TestWire_ReviewKeysReachTheEnv` drops `fix_start` from its seeded keys, its doc line and its `Env.FixStart` assertion.

Docs: `internal/burlerengine/doc.go` drops the `RunOpts.FixStart` paragraph and states the one start order; `docs/overview.md` drops `fix_start` from loom's config-module key list.

**ImpactSummary:** Removes one config key, one RunOpts field and one Env/ReviewSettings field across burlerengine, loomengine, shedrecipe and loomcli; the default parallel order every hub runs is unchanged.
