```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I tried to find a case where F1's or F2's proposed change is wrong or breaks something elsewhere in the plan.

no findings

- F1: the only read of `testMapRun.inert` is `Inert: r.inert` at `internal/gatecli/testmapplan.go:148`, which sits inside the `NewSelector` call that card 1 deletes. No test builds `testMapRun` with an `inert:` field, and no other plan card mentions `inert`. Card 4's dirty check could still reach the inert paths through `r.cfg`, the field card 1 adds, so dropping `inert` takes nothing that check needs.
- F2: `mapSideBySide` takes its tier from `mappedTierConfigs`, which returns only tiers that `cfg.Tier` resolves. That makes each tier a declared one, and its `Parallel` is at least 1. With the `Parallel` 0 row gone, the four other `TestMapSideBySide` rows still cover the clamp behaviours.

Tool calls:
1. Read `internal/gatecli/testmapplan.go` lines 130–155, grepped the plan cards for `inert`, and read `mappedTierConfigs`. My context had only the grep line numbers, not the code around them.
2. Grepped `internal/gatecli` for `inert:` literals and card 4 for its dirty-check wording, to check whether any test or later card relies on the dropped field.
