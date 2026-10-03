---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/plan/34-loomcli-wiring-reflect.md
    summary: Cards 34 and 35 delete wiring tests whose value, identity and non-Env assertions the reflective NilSeams walk over shedrecipe.Env cannot carry, contrary to D8
  - id: F2
    severity: BLOCKING
    location: _lyx/plan/39-lyxbin-kit.md
    summary: Card 39 tells a once-per-binary caller to wrap a t.TempDir-backed Build in sync.Once, so the cached binary is deleted when the first test ends and later loomcli smoke tests get a dangling path
  - id: F3
    severity: MEDIUM
    location: _lyx/plan/39-lyxbin-kit.md
    summary: New non-test spawn sites in gitkit/query.go (card 2) and testkit/lyxbin/lyxbin.go (card 39) fail TestSpawnObservability_ProductionSpawnsAreLogged, and neither card says how they pass
  - id: F4
    severity: MEDIUM
    location: _lyx/plan/36-logcapture-kit.md
    summary: Cards 20, 22, 31, 32, 36, 37 and 39 delete helpers whose callers sit in files missing from their target lists, so each card's declared footprint is wrong and the card does not build on its own
  - id: F5
    severity: MEDIUM
    location: _lyx/plan/01-testkit-root.md
    summary: The kit-on-kit importer exemption lets any kit import lyxbin and so spawn go build transitively from Tier 1 tests, and card 1 states no bound for that
  - id: F6
    severity: MEDIUM
    location: _lyx/plan/11-fabricengine-sentinels.md
    summary: Card 11 misdescribes CloneHub's one-argument refusal as a prefixed wrap and names warpbinding_test.go as owner of a remedy string that test never sees, leaving TestCloneHub_UnboundWeftNamesTwoArgForm's pin unaddressed
  - id: F7
    severity: LOW
    location: _lyx/plan/11-fabricengine-sentinels.md
    summary: Card 11's ImpactSummary gives no caller blast radius for validateWorktreeSlug, whose destroy.go caller flattens the error into a string so ErrInvalidSlug identity does not survive there
  - id: F8
    severity: LOW
    location: _lyx/plan/37-envelope-kit.md
    summary: Cards 18, 36, 37 and 39 bundle an independently testable kit with its multi-package conversion, unlike kit-only cards 19, 24 and 30
  - id: F9
    severity: LOW
    location: _lyx/plan/18-locationkit.md
    summary: Cards 18, 36, 37 and 39 do not state the package doc D11 requires for their kit
  - id: F10
    severity: LOW
    location: _lyx/plan/19-shuttlefake-kit.md
    summary: Cards 19 and 36 make assertions that only go test -race can exercise, but nothing in the plan runs -race
---

### [BLOCKING] Cards 34 and 35 delete assertions the reflective test cannot carry

**Location:** `_lyx/plan/34-loomcli-wiring-reflect.md`, `_lyx/plan/35-battencli-wiring-reflect.md`

**Issue:**
D8 replaces only the hand-picked nil-check tests: "tests that assert values or identity stay."
`envkit.NilSeams(c.env)` only reports nil fields reachable from `shedrecipe.Env`.
Several tests card 34 lists for deletion assert more than that (read in `internal/loomcli/wiring_test.go` and `wiring_commitstatus_test.go`):

- `TestWire_WebsterDepsFullyPopulated` requires `deps.Geom.WebsterDir != ""`, which is a string, and `c.env.WebsterDeps.Geom == c.runDeps.Geom`, which is an identity check.
- `TestWire_LandingSeamFieldsPopulated` nil-checks `c.registry` and `c.runner`, which are `loomCLI` fields and not part of `Env`, and compares `c.landingCfg` against `landingshed.LoadConfig` with `reflect.DeepEqual`.
- `TestWire_DiscussionSeamsFilled` requires `c.env.Shuttle == c.runner`, an identity check.
- `TestWire_DescribeSeamsFilled` requires `c.env.DescriptionPath == summaryparser.Path(loomengine.LandingDir(loc))`, a value check.
- `TestWire_ReviewSegmentSeamsFilled` requires `StencilsDir` and `RunRoot` to equal their accessors, which are value checks.
- `TestWire_CommitStatusFilled` (loomcli) checks `c.shedPaths.CommitStatus`.
  That field is on `shedbuild.ShedPaths` (`internal/loomcli/cli.go:53`), which is outside `shedrecipe.Env`.

Card 35 has the same flaw: battencli's `TestWire_CommitStatusFilled` also checks `c.shedPaths.CommitStatus`.
Deleting these tests as written loses assertions the reflective test never makes.

**Fix:**
In card 34, delete only the tests that nil-check `Env` fields alone: `TestWire_WebsterRunIsFilled`, `TestWire_PlanSeamsFilled`, `TestWire_ReflectFrictionIsFilled` and `TestWire_SkipPlanReviewIsFilled`.
From `TestWire_WebsterDepsFullyPopulated`, `TestWire_DiscussionSeamsFilled`, `TestWire_DescribeSeamsFilled` and `TestWire_ReviewSegmentSeamsFilled`, drop only the nil checks `NilSeams` now covers and keep every value and identity assertion.
Keep `TestWire_LandingSeamFieldsPopulated` and `TestWire_CommitStatusFilled` whole.
In card 35, replace only `TestWire_SeedChildClosuresFilled` and keep `TestWire_CommitStatusFilled`.

### [BLOCKING] Card 39's sync.Once guidance leaves cached binaries dangling

**Location:** `_lyx/plan/39-lyxbin-kit.md`

**Issue:**
`Build` writes into `t.TempDir()`.
Card 39 says "A caller that builds once per test binary keeps its own `sync.Once` around `Build`".
The first test's `t.TempDir()` is removed when that test ends, so every later test receives the cached path to a deleted binary.
`internal/loomcli/smoke_test.go`'s `buildLyxBinary` builds once into `os.MkdirTemp` for exactly this reason, and it records the build error instead of fataling inside the `Once`.
loomcli's smoke tests are D9-protected live tests.
They skip when tmux or claude is missing, so the breakage can pass the card's own `Verify` without anyone seeing it.

**Fix:**
Give the kit a non-fataling entry point for callers that cache: `BuildInto(dir, ldflags string) (string, error)` builds into a directory the caller owns, and `Build`/`BuildWithLDFlags` wrap it over `tb.TempDir()` with `tb.Fatalf`.
loomcli's once-per-binary caller calls `BuildInto` on an `os.MkdirTemp` directory inside its `sync.Once` and keeps recording the error the way it does today.

### [MEDIUM] New non-test spawn sites trip the spawn-observability guard

**Location:** `_lyx/plan/02-gitkit-query-helpers.md`, `_lyx/plan/39-lyxbin-kit.md`

**Issue:**
`cmd/lyx/spawnobservability_test.go` fails any non-test `.go` file under `internal/` that contains a real `exec.Command` call but neither imports `internal/logger` nor has a `spawnObservabilityAllowedSpawners` row.
`internal/gitkit/gitkit.go` is allowlisted, but a new `internal/gitkit/query.go` is not, and gitkit cannot import `logger` under its Leaf Invariant.
Card 2's `Git` helper "runs `git <args>`", and nothing stops the implementer from calling `exec.Command` in `query.go`.
`internal/testkit/lyxbin/lyxbin.go` runs `go build`.
`CONSTRAINTS.md` puts test-fixture machinery outside the Live-Substrate Spawn Observability rule, and the allowlist records such outsiders as rows, as it does for `internal/hubforge/hub.go`.
Card 39 neither adds that row nor lists `cmd/lyx/spawnobservability_test.go` as a target.

**Fix:**
Card 2: the one spawning primitive goes in `gitkit.go`, beside `MustRun` and already allowlisted, as an Edit target, and `query.go` builds every helper on it without importing `os/exec`.
Card 39: add `cmd/lyx/spawnobservability_test.go` as a target and give `internal/testkit/lyxbin/lyxbin.go` a "not governed: a test-fixture builder" row, matching the hubforge row.
That row is the only change to the guard.

### [MEDIUM] Target lists miss the callers of deleted helpers

**Location:** `_lyx/plan/36-logcapture-kit.md` (and cards 20, 22, 31, 32, 37, 39)

**Issue:**
The spec makes a card's target list its declared footprint, and the dependency graph and parallel scheduling are derived from it.
Several cards delete a helper but list only the file that defines it, not the files that call it.
Grepping HEAD shows these unlisted callers:

- Card 20: `websterengine/parentmerge_test.go` (`recordFakeEngine`).
- Card 22: `shuttleengine/run_resume_test.go` (`newTestRunner`), plus `gate_test.go`, `gate_list_test.go`, `gate_pending_test.go` and `attach_asking_test.go` (`newWaitTestRunner`/`newAttachTestRunner`).
  Card 23 does list these files, but card 22 runs first and would leave the package uncompiled.
- Card 31: `shedbuild/build_test.go`, `build_engines_test.go`, `routing_test.go` and `newshed_test.go` (`newTestEnv`).
- Card 32: `loomrecipe/sequence_test.go` and `resume_test.go` (`fakeLoomShuttle`, `fakeLoomBurler`).
- Card 36: reedengine's `attach_test.go`, `panebin_test.go`, `ensuresession_integration_test.go`, `launchscript_cleanup_test.go`, `naming_integration_test.go`, `namerepair_test.go`, `reconcile_test.go` and `spawn_test.go`; `shedadapters/burler_test.go`; and landingshed's `verifygate_test.go` and `publish_verify_test.go` (`captureLogOutput`).
- Card 37: fabriccli's `merge_cli_integration_test.go`, `mergein_webster_integration_test.go`, `status_mergeinprogress_integration_test.go` and `pushbypass_integration_test.go` (`decodeResult`), and `loomcli/parity_test.go` (`decodeSingleEnvelope`).
- Card 39: reedcli's `smoke_attach_test.go`, `smoke_coldstart_test.go`, `smoke_dotfill_test.go`, `smoke_lifecycle_test.go`, `smoke_removebyname_test.go`, `smoke_statuslineseed_test.go` and `smoke_warmpath_test.go`, and loomcli's `smoke_bootstrapwiring_test.go`, `smoke_driverstrand_test.go`, `smoke_drivingsurface_test.go` and `smoke_starttail_test.go` (`buildLyxBinary`/`buildLyxBinaryWithLDFlags`).

**Fix:**
Add each caller file to its card's `**Edit:**` list.

### [MEDIUM] The kit-on-kit exemption reaches os/exec through lyxbin

**Location:** `_lyx/plan/01-testkit-root.md`

**Issue:**
D3's importer rule reads "packages under `internal/testkit/` are imported only from `_test.go` files".
Card 1 exempts every file under `internal/testkit/` so that one kit can build on another.
The only bound it states is "every kit is itself reachable only from tests", which keeps production code out but does nothing about spawning.
The spawn-import rule checks only a kit's direct imports.
So with the exemption, `envkit` or any other kit could import `internal/testkit/lyxbin` and run `go build` from a helper that untagged tests call.
Tier Purity's `lyxbin.` token only sees the textual `lyxbin.` in test files, so it would not catch that.
The card's "What this lets through" line covers the shared-fake reversal and is silent on the exemption.

**Fix:**
Bound the exemption in the scan: no kit other than `lyxbin` itself may import `internal/testkit/lyxbin`.
Add that as a fourth plant in card 1's proof, state it in the `CONSTRAINTS.md` section, and extend "What this lets through" to name the exemption and both of its bounds.

### [MEDIUM] Card 11 assigns the clone remedy string to the wrong owner

**Location:** `_lyx/plan/11-fabricengine-sentinels.md`, `_lyx/plan/12-fabriccli-dedupe.md`

**Issue:**
`CloneHub`'s one-argument path (`internal/fabricengine/clone.go:209-213`) discards the error from `resolveEffectiveWarpURL`.
It then returns a separate `fmt.Errorf("weft %s has no recorded warp binding; ... lyx fabric clone <weft-url> <warp-url>", opts.WeftURL)`.
That is a distinct message, not a prefix on the engine refusal.
`warpbinding_test.go` tests `resolveEffectiveWarpURL`'s message (`warpbinding.go:118`) and never sees the CloneHub string.
The CloneHub string is the one the CLI emits and `cli_test.go` pins.
Its only engine pin is `TestCloneHub_UnboundWeftNamesTwoArgForm` in `warpbinding_clone_integration_test.go`, which asserts both `"has no recorded warp binding"` and the two-argument remedy.
Card 11 neither converts nor keeps that test's remedy pin.
Card 12 then says "card 11 made the engine's `warpbinding_test.go` their single owner", which is wrong for the string the CLI emits.

**Fix:**
Describe the one-argument path accurately: `clone.go:213` builds its own message, and the new error value must match `ErrNoWarpBinding` while keeping that exact text.
Name `TestCloneHub_UnboundWeftNamesTwoArgForm` as the single owner of the CloneHub remedy string: it keeps its remedy pin and replaces its `"has no recorded warp binding"` check with `errors.Is`.
Leave `warpbinding_test.go` as owner of `resolveEffectiveWarpURL`'s two strings.
Make card 12 point the CLI's dropped pins at the integration owner.

### [LOW] Card 11's ImpactSummary has no caller blast radius

**Location:** `_lyx/plan/11-fabricengine-sentinels.md`

**Issue:**
`validateWorktreeSlug` has four production callers: `add.go:79`, `cleanup.go:403`, `remove.go:97` and `destroy.go:748`.
Three of them return the error unchanged.
`destroy.go:748` flattens it into `destructiveRefusal{Reason: err.Error()}`, so `ErrInvalidSlug` identity does not survive that path.
`resolveEffectiveWarpURL` has two callers, both in `clone.go`.
No production code matches these messages by string or uses `errors.As` on them.
The ImpactSummary states none of this; it only repeats that the text stays identical.

**Fix:**
Rewrite the ImpactSummary to give that conclusion.

### [LOW] Kit creation and conversion share one card

**Location:** `_lyx/plan/18-locationkit.md`, `_lyx/plan/36-logcapture-kit.md`, `_lyx/plan/37-envelope-kit.md`, `_lyx/plan/39-lyxbin-kit.md`

**Issue:**
Each kit is independently testable and comes with its own test file.
The rubric says such a unit gets its own card, and cards 19, 24 and 30 already split kit from conversion.
Cards 18, 36, 37 and 39 bundle the kit with every consumer conversion, so the kit cannot be reviewed apart from the conversions.
Card 38 also ends up depending on the whole of card 37.

**Fix:**
Split each of the four into a kit card followed by a conversion card, and renumber the cards after them.

### [LOW] Four kit cards omit the D11 package doc

**Location:** `_lyx/plan/18-locationkit.md`, `_lyx/plan/36-logcapture-kit.md`, `_lyx/plan/37-envelope-kit.md`, `_lyx/plan/39-lyxbin-kit.md`

**Issue:**
D11 requires each kit's package doc to describe the kit.
Cards 19, 24 and 30 say what their package doc states.
Cards 18, 36, 37 and 39 say nothing about theirs.

**Fix:**
State each kit's package doc: what it builds or fakes, and what it imports, plus lyxbin's exemption bound for card 39.

### [LOW] Race-only assertions are never run under -race

**Location:** `_lyx/plan/19-shuttlefake-kit.md`, `_lyx/plan/36-logcapture-kit.md`

**Issue:**
Card 19's concurrent `AddStrand`/`Status` test and card 36's concurrent write/`String()` test exist to pass under `go test -race`.
No card `Verify` and no plan-level `verify:` command runs `-race`.
Without it, the race detector never runs, so these tests can pass even with a data race.

**Fix:**
Add a per-card `**Verify:**` running `go test -race` on each kit package.

## Focus verdicts

- Card 1 and D1/D3: the `envelope` `RequireOK`/`RequireErr` pair is licensed.
  The Scope takes #324 §6 in full, and #324 §6 names `Decode`, `RequireOK` and `RequireErr` for the decoder D2 moves into `internal/testkit/envelope`.
  Card 1's text says which helper pairs review admits.
  The kit-on-kit exemption is in D3's spirit, since kits stay transitively test-only, but it is unbounded on spawning (F5).
- Card 3 and D4: matching `gitkit.` followed by an exported identifier stays within D4.
  Any reference to a gitkit symbol from outside the package must name an exported identifier, so the narrowing drops only non-references such as the file name `gitkit.go`.
  It lets no spawn through, and the Intent says what it excludes.
  Not a defect.
- Granularity: kit-only versus kit-plus-conversion is F8.
  Multi-package mechanical conversions (cards 6, 28, 36, 37) still leave the suite green after each card, and `go test` reports failures per package.
  I do not raise them separately against D12.
- Test deletions: card 12's help-prose and conflict-envelope deletions, card 13's slug replays and card 16's watchdog deletions each match #325 §5.
  Card 12 meets D9's exception by moving any unique assertion into the kept owner.
  `livestate_refusal_selftest_test.go` pins `invalid slug ".."`, a refusal reason and not a D7 remedy string, so keeping it does not conflict with card 11.
  Cards 34 and 35 are F1.
- D11 docs: F9.
- ImpactSummary: card 8's is adequate, since hubforge has no production importer and `AddPair`'s test callers see the same behaviour.
  Card 11's is F7.
- Fabric Vocabulary: the kit-conventions Shared Decision carries the constraint to cards 18, 24 and 30.
  The structs those kits fill have no `weft`/`warp` field names, and the copies they replace are free of that vocabulary, so it has no concrete site to bite.
  Not a defect.
- `-race`: F10.
