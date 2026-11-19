# Card 9 — reach-turn

**Create:**
- `internal/shuttleengine#RerunMark.On`
- `internal/verifytree#ReachPending`
- `internal/verifytree#ReachTurn`
- `internal/verifytree#WithReachTurn`
- `internal/verifytree#ReachListedPath`
- `internal/verifytree#TestWithReachTurn`
- `internal/gatecli#ReachPendingOf`
- `internal/weaverengine_test#TestBatchGateSpec_ReachTurnOnTheBatchEntry`

**Edit:**
- `internal/shedrecipe#Env`
- `internal/shedrecipe#resolveGateEntry`
- `internal/weaverengine#Geometry`
- `internal/weaverengine#batchGateSpec`
- `internal/loomcli#loomCLI.wire`
- `internal/testkit/envkit#nilLegal`
- `internal/shedrecipe#TestResolveGateSpec_ValidOneEntryPerVocabularyName`
- `internal/gatefixer#TestWrapRowGate`
- `internal/gatefixer#scriptedGate`
- `internal/gatecli#TestGateTestReachStep`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestFullEnv_PathFieldsAbsoluteUnderOneRoot`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/testkit/envkit#TestSeamMarkers_ReflectiveTestsAreMarked`

**Extends:**
- `internal/shuttleengine/doc.go`
- `internal/verifytree/doc.go`
- `internal/shedrecipe/doc.go`
- `internal/weaverengine/doc.go`
- `internal/gatecli/doc.go`
- `internal/loomcli/doc.go`

**Uses:**
- `internal/shuttleengine#RerunMark`
- `internal/gatecli#ReadReachReport`
- `internal/gatecli#ReachReportMatches`
- `internal/gatecli#ReachReport.NoteLines`
- `internal/loomshed#NewVerifyGate`
- `internal/weaverengine#batchGateClosure`
- `internal/weaverengine#batchGateAttempts`
- `internal/gatefixer#newRowHarness`

**Intent:**
Carry the reach note to a writer whose gate runs after its turn through `verifytree.Verify` (D3's reach turn):
the loomshed verify gate of Darn-Write and Code-Burler-Fix, and the weaver's batch gate.

`shuttleengine.RerunMark` gains `On()`, which reports whether a `Run` is in progress; a nil mark's `On` is false.
The decorator below is its first caller.

`verifytree` gains the turn as a decorator over any verify gate:
- `ReachPending` is what a matching report asks of the writer: `Note`, the note's lines joined, and `Functions`, each unreached function's id (`<file>#<Func>` within its package import path) to its text hash.
- `ReachTurn` holds `Pending func() (ReachPending, bool, error)` (false when no report of the gate's key matches HEAD or it lists no unreached function),
  `Attempts`, the gate entry's attempt budget, `Rerun *shuttleengine.RerunMark` (nil where nothing re-runs the gate), and `ListedPath`.
- `ReachListedPath(verifyDir, site)` names the file in the gate's verify directory that records the functions each turn of that site listed, with their hashes; the file name derives from the site label through one named constant.
- `WithReachTurn(inner, turn)` returns a gate that runs `inner` and then:
  - a failing result of `inner` adds one to its count of failing arrivals and is returned as it is;
  - a passing result reads `Pending`. With a pending report holding a function whose id and hash the listed file does not hold,
    and with the count plus one below `Attempts`, it records the listed functions (with their hashes) through `internal/state`,
    adds one to the count, and returns one failing result with `WriterOnly` set whose findings are the note followed by the way forward:
    add the missing tests and commit them, or end your turn unchanged. Otherwise it returns the pass and resets the count.
  - The count moves once per shuttle arrival: a call inside `turn.Rerun.On()`, the wrap's re-run after a fixer, adds nothing to it, whether it fails or returns a turn,
    since that arrival's first, failing call already counted.
  - A `Pending` error is returned as the gate's error.
    A failure to record the listed functions is logged at Warn with the path and cause, and the gate returns inner's pass without the turn, since the note is advisory (D3).
  So a function listed once never brings another turn, the next arrival runs the verify as usual, and an arrival one failure short of the budget passes without the turn.
  Bound: the turn costs one writer turn and one gate attempt per arrival whose report lists an unreached function no earlier turn listed.

`gatecli.ReachPendingOf(location)` returns the `Pending` source for a gate key: it reads the worktree's report, and answers false for a report whose `GateKey` is another key, that judged nothing or that does not match HEAD (`ReachReportMatches` at `Fabric.HeadSHA`);
otherwise it returns the note and the report's unreached functions.
A report it cannot read, or cannot match against HEAD, also answers false, with a Warn naming the cause, and no error: the note is advisory, so it never stops a gate (D3).
So a row whose key does not list `reach` never gets a turn, since no report of its key exists.

Wiring:
- `shedrecipe.Env` gains `VerifyReach func(gateKey string) (verifytree.ReachPending, bool, error)`; nil turns the reach turn off, and `envkit`'s `nilLegal` lists it.
  `resolveGateEntry` wraps a `verify` element's closure with `verifytree.WithReachTurn` when `env.VerifyReach` is set:
  `Pending` reads it for the row's gate key, `Attempts` is the element's attempts, `ListedPath` is `ReachListedPath(env.VerifyDir, row+" gate")`,
  and a new `RerunMark` goes both into the turn and onto the entry's `Rerun`, so card 8's wrap marks its post-fixer re-run (Darn-Write's `darn.gate_fixer`).
- `weaverengine.Geometry` gains `ReachPending` of the same type, nil turning the turn off, and `envkit`'s `nilLegal` lists `WeaverDeps.Geom.ReachPending` beside the geometry's other nil-legal seams.
  `batchGateSpec` wraps the `batch` entry's closure (not the `card` entries) with `Attempts: batchGateAttempts`, no rerun mark and `ListedPath` from `geom.VerifyDir` and `batchGateSite`.
- `loomCLI.wire` sets `VerifyReach: gatecli.ReachPendingOf(location)` on the env and the same on the weaver geometry it builds through `hubgeom.WeaverGeometry`.

Tests:
- `TestWithReachTurn` (tier0) is one table over a scripted inner gate, a scripted `Pending` and a temp listed file:
  - a pass with a matching report listing an unreached function returns one failing `WriterOnly` result carrying the note and the way forward, and records the function;
  - the next arrival with the same function and hash passes; a function whose hash changed brings a turn again;
  - with `Attempts` 3 and two failing inner arrivals before, a pass passes without the turn;
  - after a pass resets the count, two failing arrivals and then a pass with a new function bring the turn again;
  - a failure, and a turn, inside `RerunMark.Run` add nothing to the count;
  - a pass with no pending report passes; a `Pending` error is returned as the gate's error;
  - a listed path held by a directory, so the record fails, returns the pass with no turn and logs the Warn.
- `scriptedGate` gains a `rerun *shuttleengine.RerunMark` and a per-call record of `rerun.On()`, so a subtest reads what each call saw; a nil `rerun` records false.
- `TestWrapRowGate` (tier0) gains a subtest in which a plain failure spawns one fixer and the inner gate's second call sees `On()` true while the first saw it false.
  It gains one more subtest composing the production order: `WrapRowGate` over a `scriptedGate` decorated with `WithReachTurn`, sharing one `RerunMark` on the entry and in the turn.
  An arrival whose inner gate passes with an unreached function returns the `WriterOnly` note to the writer and spawns no fixer;
  an arrival whose inner gate fails spawns one fixer, and its re-run, passing with a new function, returns the `WriterOnly` note with the fixer's reply, the turn's count moved once.
- `TestResolveGateSpec_ValidOneEntryPerVocabularyName` (tier0) asserts the `verify` entry carries a non-nil `Rerun` when `VerifyReach` is set and nil when it is not.
- `TestGateTestReachStep` (tier1) gains rows for `ReachPendingOf`: the call's own key after the call answers the note and the unreached functions;
  another key, a commit after the call, a judged-nothing report and a `reach.json` that does not parse answer false with no error.
- `TestBatchGateSpec_ReachTurnOnTheBatchEntry` (tier1) builds the spec over a committed `gitkit` worktree, with `Geometry.VerifyDir` in a `t.TempDir()` outside that worktree so the verify finds no untracked path,
  a `GateConfig` returning the template's keys and a stand-in `lym` first on `PATH` that exits 0 (so the test is not parallel: it sets the process's `PATH`),
  with `ReachPending` answering one unreached function: the `card` entry passes, the `batch` entry returns one `WriterOnly` failure carrying the note, and its next arrival passes.
- The seam-marked wiring and envkit tests listed under Edit keep passing with the new `Env` and `Geometry` fields.

Docs: `shuttleengine/doc.go` gains `RerunMark.On`; `verifytree/doc.go` gains the turn and its bound; `shedrecipe/doc.go` and `weaverengine/doc.go` gain the wrapping; `gatecli/doc.go` gains `ReachPendingOf`; `loomcli/doc.go` gains the wiring.

**ImpactSummary:** The verify gates of the rows whose key lists `reach` (Darn-Write, Code-Burler-Fix) and the weaver batch entry gain the decorator; a nil `VerifyReach` or `ReachPending` leaves every gate as it is, and the seam-marked tests over `Env` and `Geometry` are listed.
