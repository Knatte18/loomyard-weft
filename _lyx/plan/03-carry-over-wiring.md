# Card 3 — carry-over-wiring

**Edit:**
- `internal/shedrecipe#Env`
- `internal/shedrecipe#bouncerEntry`
- `internal/shedrecipe/entries_bouncer_test.go`
- `internal/shedrecipe/seam_enforcement_test.go`
- `contracts/recipes/loom-recipe.yaml`
- `internal/loomrecipe/bouncerrows_test.go`
- `internal/testkit/envkit#FullEnv`
- `internal/loomengine/discussionpath_test.go`
- `internal/loomcli#loomCLI.wire`
- `internal/battencli/wire_seams_test.go`
- `contracts/specs/shed-recipe-spec.md`

**Create:**
- `internal/loomengine#DiscussionDecisionRecordRel`
- `internal/loomcli#writeCarryOver`
- `internal/loomcli/wiring_carryover_integration_test.go`

**Uses:**
- `internal/discussionparser/carryover.go`
- `internal/shedadapters/bouncercarryover.go`

**Intent:**
Wire the carry-over seam for all three review segments, so the entry is committed by the end of each segment's `Done` transition (D2).

- `shedrecipe.Env` gains `CarryOver func(discussionparser.CarryOver) error`, a run-wide seam.
- `bouncerEntry` reads a new optional Config key, `carry_over`, whose value is the segment name the entry is filed under.
  Absent leaves `BouncerConfig.CarryOver` nil, as `commit_seam` does; present requires `env.CarryOver` through `requireSeam` and fills `Segment` with the value, `AnchorPath` with `env.AnchorPath` and `CarryOver` with `env.CarryOver`.
  The key joins `configRejectUnknown`'s list, and `seam_enforcement_test.go`'s import allowlist gains `internal/discussionparser`.
- `loom-recipe.yaml` sets `carry_over: Discussion-Review`, `carry_over: Plan-Review` and `carry_over: Webster-Review` on the three Bouncer rows, with a comment saying the value is the row's own segment.
  `bouncerrows_test.go` pins that every Bouncer row of the shipped recipe carries `carry_over` equal to its `segment`.
- `envkit`'s `FullEnv` fills `CarryOver` with a no-op closure, so recipe tests over the real rows still build.
- `loomengine` gains `DiscussionDecisionRecordRel()` in `config.go`, beside `DiscussionDirRel`, the anchor-relative form of `DiscussionDecisionRecord`, so the commit pathspec names no segment loomengine owns; `discussionpath_test.go` pins it.
- `loomcli`'s `wire` fills `Env.CarryOver` with a closure over a new `writeCarryOver(location, entry)` in `wiring.go`: `discussionparser.WriteCarryOver` on the decision record, then `fabricengine.CommitAnchoredPaths` with the decision record alone as the pathspec and the message `loom: review carry-over for <slug>`.
  No existing seam's pathspec widens; an unchanged record commits nothing, and a write or commit error is the seam's error.
  loomcli's `TestWire_EverySeamFilled` covers the fill without change.
- batten's `wire` leaves `Env.CarryOver` nil, so `battencli`'s `wire_seams_test.go` lists `CarryOver` in `intentionallyNil` as a loom-only seam (batten runs no review segment); without it that package's `TestWire_EverySeamFilled` fails.
- `shed-recipe-spec.md` documents `carry_over` beside `commit_seam` and `skip_seam`: a Config scalar naming the segment, which selects `Env.CarryOver`, with the same absent and present rules.

`entries_bouncer_test.go` covers the key: absent leaves the seam unset, a present key with a nil `env.CarryOver` is a construction error, and a present key reaches the Bouncer, which calls the seam with that segment on a `CONVERGED` settle.
`wiring_carryover_integration_test.go`, tagged `integration`, drives `writeCarryOver` over a hub fixture built through `internal/hubforge` and asserts the entry lands in the record and the record is committed alone, other dirty files staying uncommitted.

Bound: the key only selects an `Env` seam; a recipe without it builds every Bouncer exactly as before.

**ImpactSummary:** All three shipped Bouncer rows now write and commit a carry-over entry on Done; non-loom recipes and rows without the key are unchanged.
