# Card 11 — harden-rows

**Create:**
- `plan:internal/hardenshed#Campaign` -> `type Campaign struct`
- `plan:internal/hardenshed#Campaign.LiveGate` -> `func (c *Campaign) LiveGate(runDir string) shuttleengine.Gate`
- `plan:internal/hardenshed#Campaign.WrapRow` -> `func (c *Campaign) WrapRow(inner shedengine.ShedProducer, row string, kind RowKind, runDir string) shedengine.ShedProducer`
- `plan:internal/hardenshed#Campaign.RoundFacts` -> `func (c *Campaign) RoundFacts(runDir string, round int) (string, error)`
- `plan:internal/hardenshed#Campaign.LiveRed` -> `func (c *Campaign) LiveRed(runDir string, round int) (bool, error)`
- `plan:internal/hardenshed#Campaign.Halt` -> `func (c *Campaign) Halt(row string) error`
- `plan:internal/hardenshed#Campaign.ArchiveFinal` -> `func (c *Campaign) ArchiveFinal(runDir string) error`
- `plan:internal/hardenshed#TestCampaignRoundFacts` -> `func TestCampaignRoundFacts(t *testing.T)`
- `plan:internal/shedrecipe#hardenFixEntry` -> `func hardenFixEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `plan:internal/shedrecipe#TestHardenFixEntry` -> `func TestHardenFixEntry(t *testing.T)`
- `plan:internal/shedrecipe#TestHardenRoundKey` -> `func TestHardenRoundKey(t *testing.T)`

**Edit:**
- `internal/shedrecipe#registry`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#bouncerEntry`
- `internal/shedrecipe#burlerReviewEntry`
- `internal/shedrecipe#TestBouncerEntry_CommitSeam`
- `internal/shedrecipe#TestBurlerHalfEntries_StrictUnknownKeys`
- `internal/shedrecipe_test#coverageGuardAllowedUnreachableEngines`
- `internal/testkit/envkit#nilLegal`
- `internal/testkit/envkit#FullEnv`
- `internal/shedrecipe/doc.go`

**Extends:**
- `internal/hardenshed/doc.go`

**Uses:**
- `internal/shedrecipe#burlerFixEntry`
- `internal/shedrecipe#resolvePlainGateClosure`
- `internal/shedadapters#NewBurlerFixProducer`
- `internal/burlerengine#ParseFixerReport`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestFullEnv_PathFieldsAbsoluteUnderOneRoot`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/testkit/envkit#TestSeamMarkers_ReflectiveTestsAreMarked`

**Intent:**
Build the Harden-Review ring's rows (D3, D6, D7).

`hardenshed.Campaign` bundles one run's loaded `Config`, the live, reap, row and archive seams of cards 6 to 9, and the round models; card 14 builds it.
Its methods hand the rows what they need per run directory:
- `LiveGate` is `NewLiveGate` over that directory.
- `WrapRow` is `WrapRow` with the campaign's seams.
- `RoundFacts` renders the Hardener section of the round's facts: the round's reviewer and fixer models, the live report's mark, runs and started counts, the stray report, the fix commits, and the deferred findings, those the round's fixer report defers, read through `burlerengine.ParseFixerReport`, and those a red live report lists. A file the round lacks is named as absent, never an error.
- `LiveRed` reads the round's live report through `ReadLiveReport`; an absent report is not red.
- `Halt` reaps at `ReapHalt` and appends to the last round's stray report.
- `ArchiveFinal` is `ArchiveFinalLedger`.
`TestCampaignRoundFacts` is tier0 over a run directory holding each file, over one whose live report is red with deferred failures and whose fixer report defers a finding, both deferred sets rendered beside the red mark, and over one lacking the files.

`Env` gains `Harden *hardenshed.Campaign`, nil for every recipe but hardener; `envkit`'s `nilLegal` gains `Harden` and `FullEnv` fills it, so the seam tests keep passing.

`registry` gains `"HardenFix": hardenFixEntry`.
No embedded recipe names `HardenFix` until card 13, so `coverageGuardAllowedUnreachableEngines` gains it here, with card 13 named as the card that drops it.
`hardenFixEntry` builds the `BurlerFix` producer as `burlerFixEntry` does, with these differences:
- Its gates are fixed: `clean` (3 attempts), `verify` over the hub key `harden-burler-fix` with `Env.Harden`'s `verify_attempts`, built by `resolvePlainGateClosure` so it reads `Env.VerifyCommand` and `Env.VerifyTreeScope` for that key, and `live`, `Env.Harden.LiveGate(runDir)` with `live_attempts` attempts and `PassOnCap` set, so a spent live budget ends the row instead of halting the run.
- It refuses a `gates` or `gate_key` key, naming "the HardenFix row's gates are fixed to clean, verify and live; way forward: remove the key from the recipe row", so a recipe cannot configure the live gate off.
- It refuses a profile whose `fix-scope` is not `source`.
- It refuses a nil `Env.Harden`, naming the wiring.
- It returns the producer wrapped by `Env.Harden.WrapRow(…, RowFix, runDir)`.

`burlerReviewEntry` takes an optional bool key `harden_round`; set, it refuses a nil `Env.Harden` and wraps its producer by `WrapRow(…, RowReview, runDir)`.
`bouncerEntry` takes the optional bool key `harden_round`; set, it refuses a nil `Env.Harden` and fills `BouncerConfig.RoundFacts`, `LiveRed` and `OnHalt` from the campaign over the row's run directory.
Its `commit_seam` switch gains `harden`, which requires `Env.Harden` and commits through `ArchiveFinal`; the refusal of an unknown seam names `harden` beside `plan` and `discussion`.

`TestHardenFixEntry` is tier0:
- the built row's gates are clean, verify and live in that order with the stated attempts and `PassOnCap` on live only;
- the verify gate asks a recording fake `Env.VerifyCommand` and `Env.VerifyTreeScope` for `harden-burler-fix`;
- the verify attempts are `Env.Harden`'s `verify_attempts`, whatever `Env.DarnVerifyAttempts` holds;
- a `gates` key, a `gate_key` key, a non-source fix scope and a nil `Env.Harden` are each refused with their way forward.
`TestHardenRoundKey` is tier0 over a campaign with recording fake seams:
- a review row with `harden_round` and a nil `Env.Harden` is refused, and with a campaign its producer's call reaches the campaign's reap at the start and the end;
- a Bouncer row with `harden_round` and a nil `Env.Harden` is refused, and with a campaign its rendered judge prompt's facts file holds the campaign's section.
`TestBouncerEntry_CommitSeam` gains the `harden` row and the nil-campaign refusal; `TestBurlerHalfEntries_StrictUnknownKeys` admits `harden_round` on the review row.
The shedrecipe package doc gains the `HardenFix` row and the `harden_round` keys.

**ImpactSummary:** One new registry engine, two optional row keys and one Env field listed nil-legal in envkit; rows that set none of them build as before.
