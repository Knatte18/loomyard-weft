# Card 1 — adopt-pause-caller

**Create:**
- `internal/pauserecord#AdoptCaller`

**Edit:**
- `internal/pauserecord#Record.CoverageOf`
- `internal/pauserecord#Coverage`
- `internal/pauserecord#HoldCaller`
- `internal/pauserecord#TestCoverageOf`
- `internal/pauserecord/doc.go`
- `internal/dobbyengine#RunView`
- `internal/dobbyengine#planPass`
- `internal/dobbyengine/doc.go`
- `internal/hibernateengine#TestSequence`

**Extends:**
- `internal/tenderengine/doc.go`

**Intent:**
D5's adopt pause gets its own caller, and every reader of the old adopt hold reads it the same way.

`pauserecord.AdoptCaller` (`"adopt"`) is the caller of the entry `lym adopt`'s `pause` step writes.
`HoldCaller` stays as the legacy caller an older adopt left; its comment says so, and that only `lym tender resume` or `lym batten cancel` lifts it now.
Its comment's "Only the green adopt check of the entry's subject lifts it by itself" goes, and so does the pauserecord `doc.go` sentence "Such an entry holds until its subject's adopt check is green";
the doc names `AdoptCaller` beside `HoldCaller`, both lifted only by hand.
`Record.CoverageOf` sets `Coverage.Held` when a live entry of the slug's own scope carries `AdoptCaller` or `HoldCaller`;
the field comment says both callers set it.
`battencli.DobbyRuns` copies `Coverage.Held` into `dobbyengine.RunView.Held` unchanged, so the dobby's existing held treatment covers the adopt pause with no dobby code change:
no pass of any kind, no owner-pause recheck (so the dobby's owner lift never lifts it), and the full pass at the lift.
A released task under the pause is covered, so `heldByEntry` already starts none until the lift.
The hibernate wake only adds `OwnerCaller` entries and lifts none, so it leaves the adopt pause standing.
`RunView.Held`'s comment, the hold-caller sentence of `planPass`'s comment and the dobbyengine `doc.go` passage on hold-caller entries are reworded to the adopt pause, legacy `adopt-hold` included, lifted only by hand.

Tests, tier0:
- `TestCoverageOf` gains rows: an own-scope `AdoptCaller` entry sets `Held` and leaves `OwnerHeld` unset, and an own-scope `HoldCaller` entry still sets `Held`.
- `TestSequence` (tier0) gains a row beside the owned-elsewhere wake row: a live `AdoptCaller` entry of a pair slug stands after the wake, unchanged, beside the wake's own entries.

`lym tender pause` and `lym tender resume` need no code or new row: `Pause` answers `AlreadyPaused` over any live entry of the scope, whatever its caller, and `Resume` lifts every live entry of the slug's scope.
`TestPause`'s row "a request's Caller labels the added entry and adds none over a live entry of the scope" and `TestResume`'s slug-entry rows already pin both for a caller other than the operator's.
The tenderengine `doc.go` gains one sentence: an adopt pause is an operator pause, which `Resume` of its slug lifts and a `Pause` of its slug answers already paused.

**ImpactSummary:** one new constant and one widened coverage flag in a leaf package; the dobby, batten cancel and tender read the flag or the scope unchanged, so only comments and test rows change outside pauserecord.
