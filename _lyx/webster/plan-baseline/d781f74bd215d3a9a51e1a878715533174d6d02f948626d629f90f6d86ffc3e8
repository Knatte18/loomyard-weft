# Card 13 — loom-wiring-fans

**Edit:**
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#TestWire_ReviewKeysReachTheEnv`
- `docs/overview.md`

**Intent:**
The wiring half of decision D1.
`loomCLI.wire` fills `Env.RowClusterFans` with `discussion_fan` for the Discussion-Bouncer and Discussion-Burler rows and `plan_fan` for the Plan-Bouncer and Plan-Burler rows, through the `loomshed` row-name constants, so each segment's Bouncer and round agree by construction.
No Webster row gets an entry: Webster-Review stays solo, since its forks may run no git and its subject exists only through git (D9).
`RowReviewModels` already carries card 12's fan-aware reviewer lists.

`TestWire_ReviewKeysReachTheEnv` covers a `loom.yaml` with both fan keys set reaching all four rows, the fanned rows' reviewer lists coming from `fan_review`, the template's empty fans leaving every entry empty, and Webster-Burler and Webster-Bouncer never present in `RowClusterFans`.
`docs/overview.md`'s burler line states that `loom.yaml`'s `discussion_fan` and `plan_fan` turn the fan on for Discussion-Review and Plan-Review, off by default.

**ImpactSummary:** One new Env field filled in wire; with the template's empty fans every row builds and runs exactly as before.
