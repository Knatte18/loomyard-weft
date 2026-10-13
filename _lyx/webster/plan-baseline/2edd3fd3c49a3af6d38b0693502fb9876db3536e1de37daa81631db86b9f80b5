# Card 7 — darn-hard-cap-config

**Edit:**
- `internal/darnengine#Config`
- `internal/darnengine#ConfigSpec`
- `internal/darnengine#LoadConfig`
- `internal/darnengine/template.yaml`
- `internal/darnengine/doc.go`
- `internal/loomcli#loomCLI.wireDarn`
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`
- `internal/darnengine#TestLoadConfig`
- `internal/loomcli#TestWire_DarnRecipe`

**Intent:** darn config gains the Darn-Review perch's hard cap (D3), refused at load when it cannot hold, and wired into the Darn-Bouncer row.

`Config` gains `ReviewHardMaxBounces int` with tag `yaml:"review_hard_max_bounces"`, documented as the round number at or past which the Darn-Review perch never continues a round.
`template.yaml` adds `review_hard_max_bounces: 4` after `review_max_bounces`, commented: the Darn-Review perch's hard cap; a round numbered at or past it is accepted or escalated `hard-cap`, never continued; at least 1 and not below `review_max_bounces` while that is above 0.
`review_max_bounces`' comment changes "a spent budget escalates to the parent" to "a spent budget runs the soft-cap rule".
`ConfigSpec` classifies the key `CategoryGuard`, the layer policy of `review_max_bounces` (`PATTERN-hub-wide-config`), so it is closed to the local layer and set only in the hub file; its doc comment names both keys.

`LoadConfig`, after the writer check:

- `review_hard_max_bounces` below 1 is refused: `darn config key "review_hard_max_bounces": must be at least 1, got %d; way forward: run "lyx config darn --set review_hard_max_bounces=<n>" from the prime, then "lyx loom resume"`.
- With `review_max_bounces` above 0 (0 skips the review, D3), a hard cap below it is refused, naming both keys and values.
  The way forward: raise the hard cap with `lyx config darn --set review_hard_max_bounces=<n>` or lower the soft cap with `lyx config darn --set review_max_bounces=<n>`, from the prime, then `lyx loom resume`.
  darn's budget keys are barred from the local layer, so the hub file is the only route.

Its doc comment and `internal/darnengine/doc.go` name the new key, its default of 4 and both refusals.

`loomCLI.wireDarn` sets `c.env.RowHardMaxBounces[loomshed.NameDarnBouncer] = cfg.ReviewHardMaxBounces` into the map `loomCLI.wire` filled for the loom rows, creating the map when it is nil.

`refusal-spec.md` gains, beside "darn.yaml: review_max_bounces below 0", rows for:

- "darn.yaml: review_hard_max_bounces below 1", with the way forward above;
- "darn.yaml: hard cap below soft cap", a correctness halt raised at load when `review_max_bounces` is above 0 and above `review_hard_max_bounces`, with the way forward above.

`docs/overview.md`'s darn sentence on the perch's bounce budget adds the hard cap `review_hard_max_bounces`, default 4.
Its next sentence changes from "A spent budget escalates to the parent through the `lyx loom circling` verbs as a loom review segment does" to: a spent budget runs the soft-cap rule as a loom review segment does, escalating through the `lyx loom circling` verbs only when stuck.

Tests:

- `TestLoadConfig`'s two loading rows expect `ReviewHardMaxBounces: 4`.
  New rows: `review_hard_max_bounces: 0` refused naming `review_hard_max_bounces` and `lyx config darn --set`, and `review_max_bounces: 5` under the default hard cap refused naming both keys and `lyx loom resume`.
  A subtest gives a local `review_hard_max_bounces: 9` over the hub file and wants `configengine.ErrInvalid` naming the local file and the key, pinning the guard category.
- `TestWire_DarnRecipe`'s darn-seed subtest asserts `c.env.RowHardMaxBounces[loomshed.NameDarnBouncer]` is 4, the template's.

**ImpactSummary:** One new hub-wide darn.yaml key with two load refusals and one Env entry; existing hub files gain the key through the template, and a darn.yaml whose review_max_bounces is above 4 now refuses at load.

**Commit:** 7: darn.yaml hard cap for the Darn-Review perch
