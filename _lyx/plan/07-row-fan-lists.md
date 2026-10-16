# Card 7 — row-fan-lists

**Create:**
- `plan:internal/shedrecipe#RowModelKeys` -> `type RowModelKeys struct`

**Edit:**
- `internal/shedrecipe#Env`
- `internal/shedrecipe#burlerReviewEntry`
- `internal/shedrecipe#burlerRoundProfile`
- `internal/shedrecipe#burlerFixEntry`
- `internal/shedrecipe#bouncerEntry`
- `internal/shedrecipe#TestBurlerReviewEntry_ClusterFan`
- `internal/shedrecipe#TestBurlerHalfEntries_StrictUnknownKeys`
- `internal/shedrecipe#TestBurlerFixEntry_GateConfig`
- `internal/shedrecipe#TestBouncerEntry_ClusterExcludes`
- `internal/testkit/envkit#nilLegal`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#TestWire_ReviewKeysReachTheEnv`
- `contracts/specs/refusal-spec.md`
- `internal/shedrecipe#captureFix`
- `internal/loomrecipe#gateQuiescenceFailureMessage`
- `internal/loomrecipe#gatedRowFanViolations` -> `func gatedRowFanViolations(r shedbuild.Recipe, fans map[string][]string) []string`
- `internal/loomrecipe#TestNoGatedRowAuthorizesForkSubagents`

**Uses:**
- `internal/shedrecipe#captureReview`
- `plan:internal/shedadapters#RoundFans`

**Intent:**
Carry a segment's per-round fan lists and its model keys from the wiring to its rows (D2, D7, D10), in place of one fan per row.

`Env.RowClusterFans` becomes `map[string][]string`, a review segment's per-round reviewer fan list keyed by its Bouncer and review rows.
`Env` gains `RowFixFans map[string][]string`, the per-round fixer fan list keyed by the segment's review row, and `RowModelKeys map[string]RowModelKeys`, keyed by row name.
`RowModelKeys` holds `Review` and `Fix`, the `shuttleengine.ModelKey` each half's model came from, and `FixFan`, the name of the fixer fan key; envkit's `nilLegal` admits both new maps, whose nil is the documented absent value.

`burlerRoundProfile` recognises a ninth key, `fix-fan`, onto `Profile.FixFan`; `burlerFixProfile` still rejects it, since a fix row names no fan.
`burlerReviewEntry` builds the row's `RoundFans` when the row has a `RowClusterFans` or `RowFixFans` entry, or its recipe profile sets `cluster-fan` or `fix-fan`: each list is the wiring's entry when it holds a non-empty name, else the profile's one value, else the wiring's entry, and the profile's fans are cleared so the producer picks per round.
A wiring list whose every entry is "" counts as absent beside a recipe fan, as today's empty `RowClusterFans` entry does, so under the template defaults a recipe `cluster-fan` or `fix-fan` still runs.
A wiring list holding a non-empty name beside the recipe's `cluster-fan` keeps today's construction refusal, and one beside the recipe's `fix-fan` is refused the same way: `fix fan list %v from loom.yaml through the wiring conflicts with fix-fan %q set in the recipe's profile; way forward: remove fix-fan from the recipe's profile for this row`.
A recipe `fix-fan` is recorded by the review row but read only by a fix row told a fixer fan key, so on a Webster, Merge or Darn row it runs no twill fixer.
The row's `opts.ReviewKey` is its `RowModelKeys` entry's `Review`, replacing the hard-coded `fan_review` key, and a row with no entry keeps the zero key.
`burlerFixEntry` sets `opts.FixKey` and `opts.FixFanKey` from its `RowModelKeys` entry and sets `RequiresFans` exactly when that entry names a `FixFan` key, so only a fix row the wiring tells a fixer fan key refuses an unrecorded round.
`bouncerEntry` passes the row's `RowClusterFans` list as `BouncerConfig.ClusterFans` and stops widening its `cluster_excludes` key from the row's fan, so the recipe key alone sets `ClusterExcludes`.

`internal/loomrecipe`'s gate-quiescence test re-states its premise for the twill fixer D7 decides, which is the re-opening its message asks for:
`gatedRowFanViolations` takes `fans` as `map[string][]string`, the type `Env.RowClusterFans` now carries, a row receiving a fan when any entry is non-empty, and keeps refusing a cluster fan on every gated row, a gated `BurlerFix` row included.
`gateQuiescenceFailureMessage` and the two comments state that a turn end whose Stop payload reports an outstanding fork is a wait, so no gate runs while a fork is live; a `BurlerReview` reviewer forks under its cluster fan, and a gated `BurlerFix` fixer forks only under its segment's fixer fan, which reaches it through the review row's recorded `fix_fan`, never a cluster fan.
The backstop is the reviewer audit's exact count and the fixer audit's per-fork violation checks, the fixer's count being a warning per D9.
`TestNoGatedRowAuthorizesForkSubagents` passes its fan maps as lists and keeps every row's verdict.

`loomCLI.wire` keeps `loom.yaml`'s scalar fan keys for now, giving each Discussion and Plan Bouncer and review row the one-entry list of its segment's fan, and fills `RowModelKeys` for those two review rows with `loom`'s `fan_review` key while the segment's fan is set, today's key; card 8 replaces both with the per-round keys.

Tests:
`TestBurlerReviewEntry_ClusterFan` table-drives the lists: a `RowClusterFans` and `RowFixFans` pair reaches the producer, observed through the profile the fake runner records for round 1 (`ClusterFan` the list's first entry) and the record's `fix_fan`;
a recipe `cluster-fan` or `fix-fan` with no wiring entry reaches round 1 the same way;
each of the two conflicts is a construction error naming its way forward, and a `[""]` wiring list beside a recipe `cluster-fan` or `fix-fan` constructs and runs the recipe's fan;
the row's `RowModelKeys` entry reaches `opts.ReviewKey` and a row with none keeps the zero key.
`TestBurlerHalfEntries_StrictUnknownKeys` gains rows: `fix-fan` is accepted on a review profile and rejected on a fix profile.
`captureFix` also writes a round-1 usage record carrying `fans_recorded`, so a fix row told a fixer fan key reaches its runner, and `TestBurlerFixEntry_GateConfig` gains a row whose `RowModelKeys` entry reaches `FixKey` and `FixFanKey` through it.
`TestBouncerEntry_ClusterExcludes`'s env-fan subtests pass lists, and the one-entry list now asks for `exclude_lenses` only when round 1's record names the same fan: its `judgePrompt` helper writes a hand-built `round-1-usage.yaml` into the run directory holding `fans_recorded: true`, `fan: env-fan` and a review half, since `recordRoundFans` is unexported in `shedadapters`.
`TestWire_ReviewKeysReachTheEnv` compares `RowClusterFans` as lists and asserts the two review rows' `RowModelKeys`.

`refusal-spec.md` gains the fix-fan construction conflict row in the loom section, beside the review rows.

**ImpactSummary:** `Env`'s fan field changes type, read by the three entries and filled only by `loomCLI.wire` and mirrored by `internal/loomrecipe`'s gate-quiescence test helper; with today's scalar config every row gets the same fan and model key it has now, and the Discussion and Plan review rows now record their names.
