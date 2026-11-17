---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:guard-reach
    location: _lym/plan/06-seat-parity.md (Profile.SeatTables, seatTables) and _lym/plan/07-producer-table.md (unlistedLiterals)
    summary: The burler assist seat and the assisted chair stencil escape both the seat catalog and the scan, so D1's every-seat comparison misses the one seat Codex runs as an assistant
    fix: In card 6, change the Create glyph of Profile.SeatTables to `func (p *Profile) SeatTables(opts RunOpts, skills []string, assist *seatengine.Seat) []seatengine.Table` and state that it also returns `withAssistant(reviewTable(...), assist)` when assist is non-nil. The burler `seatTables` entry's Build passes a stand-in assist seat and lists `internal/burlerengine#Engine.assistSeat` in its Holders. Add `internal/burlerengine#withAssistant` and `internal/burlerengine#Engine.assistSeat` to card 6's Uses. In card 7, have `unlistedLiterals` also report a `seatengine.Seat` composite literal, value or pointer, that sets `Stencil` and whose enclosing function `listed` does not hold. Add a TestLaunchSpecLiterals_AreListed row for an unlisted pointer `seatengine.Seat` literal that sets `Stencil`.
  - id: F2
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/plan/06-seat-parity.md (plan:internal/shuttleparity#Seats)
    summary: PATTERN-test-only-code is broken because shuttleparity.Seats is a production function only TestSeats_CodexManifestHoldsClaudes calls, while Report goes through report and seatsOf
    fix: Drop `plan:internal/shuttleparity#Seats` from card 6's Create list and from its Intent. Have TestSeats_CodexManifestHoldsClaudes call `seatsOf(env, seatTables)` and fail on each builder error it returns.
  - id: F3
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/plan/02-manifest-record.md (plan:internal/shuttleengine#Manifest.Omitted)
    summary: PATTERN-test-only-code is broken because Manifest.Omitted, the record's verdict, has no production caller in any card; only tests in cards 2, 3 and 4 call it
    fix: In card 2's Intent, give the record a `Verdict []ManifestItem` field that WriteManifest fills from `m.Omitted()` before it writes. The verdict is then stored in `manifest.json` for the bench report to read, and Omitted is reached from every Prepare. Add a TestManifest_Record row asserting that the written record's verdict holds exactly the omitted items.
  - id: F4
    severity: MEDIUM
    class: decision
    origin: handler
    location: _lym/plan/00-overview.md (Decision a seat Codex cannot start is not compared), _lym/plan/06-seat-parity.md, _lym/plan/07-producer-table.md
    summary: Skipping every seat or producer Codex cannot start narrows D1's acceptance criterion with no decision-record entry; it covers fanned burler review and fix, Plan-Write and Discussion seats, Merriam and the loom driver
    fix: Either compare those seats too, since Prepare does not refuse a requirement and only Start does, and drop the Unmet skip from the TestSeats and TestProducers tests. Or ask the parent through `lym creel post --parent` to record the exemption with `lym loom decision add` as an Added after Discussion entry, and cite that entry in the overview's Shared Decision.
  - id: F5
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/06-seat-parity.md (newParityFixture)
    summary: The parity fixture holds one skill, but every catalog seat and producer loads its configured skills; Codex lists a skill-load item only for a skill under its skill roots, so every seat with skills shows missing items by construction
    fix: In card 6's Intent, have newParityFixture write a SKILL.md, under the fixture's codex.yaml skill_roots and the Claude config directory, for every skill named by the specs the catalog and the producer table build. Collect the names from the built specs before PrepareEach runs. State this in the TestSeats_CodexManifestHoldsClaudes bullet, and in card 7's TestProducers bullet, which reuses the fixture.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/06-seat-parity.md (PrepareEach, seatsOf)
    summary: Card 6 adds PrepareEach's Prepare-error and absent-record refusals and seatsOf's per-builder error, but only card 8's TestReport reaches them, so card 6 carries behavior its own tests do not cover
    fix: Move `plan:internal/shuttleparity#manifestEngine` from card 8's Create list to card 6's. Move three rows from card 8's TestReport into card 6's TestCompare, over manifestEngine fakes and a fixture seatTable whose Build fails. The rows are a failing Prepare naming the engine, an absent record naming the engine and shuttleengine.WriteManifest, and a failing builder whose error names its glyph while the other entry's seats still build. Card 8's TestReport keeps its report-level rows.
  - id: F7
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/01-heddle-standing-keys.md (plan:internal/shuttleengine#TestStandingPassages)
    summary: TestStandingPassages repeats the outside_hub, no_reader and reader_error rows of claudeengine TestPrepare_AppendsStandingPassages, which already pin the zero result through Prepare at the public surface
    fix: Drop `plan:internal/shuttleengine#TestStandingPassages` from card 1's Create list and its test bullet. State in the TestPrepare_AppendsStandingPassages bullet that those rows keep pinning the zero Standing. Card 3's TestPrepare_WritesTheManifest already covers a Standing carrying a file entry.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/02-manifest-record.md (Runner.launch copy paragraph)
    summary: The Intent says a start retry's failed attempt leaves no copy, but the copy is made after saveRunState and a failed awaitStartup returns through the retry loop, keeping that attempt's copy
    fix: Replace the sentence with one saying an attempt that fails after its run record is saved keeps its copy under its own run id. Name such an attempt as one whose startup the start retry relaunches. An attempt that fails before then leaves none.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/07-producer-table.md (TestProducers_CodexManifestHoldsClaudes bullet)
    summary: The test bullet names only Merriam's Run as logged and not compared, but loomcli driverSpec also sets ForkSubagents and requires RequirementBackgroundWake
    fix: Name `internal/loomcli#driverSpec` beside `internal/websterengine#Run` as a producer logged with its unmet requirements and not compared.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/02-manifest-record.md (copyManifest warning)
    summary: The copy warning gives one way forward, to remove whatever blocks the manifests directory, for both branches, which is wrong for an unreadable record
    fix: Give the unreadable-record branch its own way forward, naming the run directory's manifest.json and the engine's manifest write. Keep the directory way forward for a failed copy write.
  - id: F11
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/03-claude-manifest.md (instruction-source bullet)
    summary: Card 3 derives the repository root as the nearest directory holding a .git entry without naming dotgit.FindRoot, so it may hand-roll a second walk beside the one Codex uses
    fix: Add `internal/dotgit#FindRoot` to card 3's Uses and state that the repository root comes from it on the symlink-resolved anchor, as codexengine's readCheckout does.
  - id: F12
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/plan/07-producer-table.md (renderer list)
    summary: The renderer list names shedarbiter's and gatefixer's renderers by file, not by symbol, and the arbiter renderer lives in prompt.go, not call.go
    fix: Write `shedarbiter.Prompt` and `gatefixer.RenderPrompt` in place of the two by-file descriptions.
---

### [BLOCKING] The assist seat escapes the seat catalog and the scan

**Location:** `_lym/plan/06-seat-parity.md`, `_lym/plan/07-producer-table.md`

**Issue:** D1 compares "every seat in `seatengine`'s tables".
The burler review table gains its assistant seat and the assisted chair stencil after the literal is built:
`internal/burlerengine/assist.go:124` builds the seat as `&seatengine.Seat{`, and `withAssistant` (`assist.go:152-160`) swaps the chair's stencil to `assistedReviewerStencil` and appends the seat.
`review.go:146` and `review.go:157` call it.
Card 6's `Profile.SeatTables` returns `reviewTable` and `fixTable` bare, and card 7's scan matches only `seatengine.Table` literals.
Neither the catalog nor the scan can see this seat.
It is the seat a Codex assist trial runs on Codex, so the parity case this task exists for goes untested.

### [BLOCKING] `shuttleparity.Seats` is reached only by a test

**Location:** `_lym/plan/06-seat-parity.md`

**Issue:** `PATTERN-test-only-code` requires every production function to be reachable from a `main` package.
Card 8's `Report` calls `report(env, seatTables, ...)`, which calls `seatsOf` directly.
The only caller of `Seats` is `TestSeats_CodexManifestHoldsClaudes`.
Once card 8 imports `shuttleparity` into `shuttlecli`, `cmd/lym/testonlycode_linux_test.go` reports it.

### [BLOCKING] `Manifest.Omitted` has no production caller

**Location:** `_lym/plan/02-manifest-record.md`

**Issue:** D1 says an omitted item counts as missing in the record's verdict.
Card 2 makes `Manifest.Omitted` that verdict, but no card calls it from production code.
Card 8's verb compares through `Compare`, which uses `MissingItems`, and it never reads a run record.
The callers are `TestManifest_Record` and the Prepare tests of cards 3 and 4.
`shuttleengine` is imported by every main, so the test-only-code guard reports the method.
Storing the verdict in the record also gives the bench report of D11 something to read instead of something to recompute.

### [MEDIUM] The unmet-requirement skip narrows D1 with no recorded decision

**Location:** `_lym/plan/00-overview.md`, `_lym/plan/06-seat-parity.md`, `_lym/plan/07-producer-table.md`

**Issue:** D1's acceptance criterion holds every seat and every other producer to parity.
Its auto-mode assumption says there is no exemption table.
The plan skips every spec Codex's `Start` would refuse, through `Unmet`.
Seats do set `ForkSubagents`: `burlerengine/review.go:87` and `fix.go:97` set it whenever a fan is configured, and `loomengine/plan.go:133` and `discussiontable.go:158` set it too.
The skip therefore drops fanned burler review and fix, Plan-Write and Discussion seats, Merriam, and the loom driver.
`Prepare` itself does not refuse a requirement, so a comparison is possible.
Skipping them is a design call the decision record does not make.

### [MEDIUM] The parity fixture does not hold the seats' skills

**Location:** `_lym/plan/06-seat-parity.md`

**Issue:** Card 3 gives Claude one `skill-load` item per spec skill the load message names.
Card 4 gives Codex one only for a skill whose SKILL.md a skill root holds (`codexengine/skillload.go:41-50`, `resolvedSkillPaths`).
The builders take each seat's skills from configuration and the language skills.
`newParityFixture` writes one skill.
So every seat that loads any other skill reports a missing `skill-load` item, and the test fails whatever Codex delivers.

### [MEDIUM] Card 6's refusals are reached only by card 8's test

**Location:** `_lym/plan/06-seat-parity.md`

**Issue:** `PrepareEach` refuses a failing `Prepare` and an absent record, and `seatsOf` returns one error per failing builder.
Card 6's tests have no row that reaches any of them.
`TestSeats_CodexManifestHoldsClaudes` only fails when a real builder errors.
The rows that reach them sit in card 8's `TestReport`, through the `manifestEngine` fake card 8 creates.

### [MEDIUM] `TestStandingPassages` repeats an existing test

**Location:** `_lym/plan/01-heddle-standing-keys.md`

**Issue:** `internal/shuttleengine/claudeengine/prepare_test.go:586`, `TestPrepare_AppendsStandingPassages`, has rows `outside_hub`, `no_reader` and `reader_error`.
Through `Prepare`, they pin that an empty Bolt dir, no installed reader and a reader error each give no passages, with the Warn log.
Card 1 keeps that test, and it now returns a `Standing`.
The new table's rows assert the same three cases at a lower level.
Its last row, a reader's `Standing` coming back whole, is covered by the Prepare tests and card 3's file-entry row.

### [LOW] The copy bound misstates the start retry

**Location:** `_lym/plan/02-manifest-record.md`

**Issue:** `Runner.launch` builds the `Run` and calls `awaitStartup` after `saveRunState` (`internal/shuttleengine/run.go:731-760`).
The retry loop at `run.go:605-615` relaunches only an attempt that returned a run, which is one past `saveRunState`.
So every retried attempt has already copied its record, and the sentence saying it leaves no copy is false.

### [LOW] Card 7 omits the loom driver from the unmet producers

**Location:** `_lym/plan/07-producer-table.md`

**Issue:** `internal/loomcli/driverspec.go:62-63` sets `ForkSubagents: true` and requires `RequirementBackgroundWake`.
The test bullet names only `internal/websterengine#Run` as logged and not compared.

### [LOW] One way forward for two copy failures

**Location:** `_lym/plan/02-manifest-record.md`

**Issue:** The warning tells the operator to remove whatever blocks `<root>/manifests`.
For a run-directory record that is not JSON, nothing blocks that directory, and the cause is the engine's write.

### [LOW] Card 3's repository root has no named source

**Location:** `_lym/plan/03-claude-manifest.md`

**Issue:** Card 3 states that the repository root is the nearest directory at or above the anchor holding a `.git` entry.
Codex gets the same root through `dotgit.FindRoot` (`codexengine/home.go:74`).
Naming that source keeps the two engines keying files from one derivation, as the card intends, and avoids a new root walk in an engine.

### [NIT] Two renderers named by file

**Location:** `_lym/plan/07-producer-table.md`

**Issue:** The list says "shedarbiter's prompt renderer in `call.go`" and "gatefixer's prompt renderer in `prompt.go`".
The functions are `shedarbiter.Prompt` (`internal/shedarbiter/prompt.go:71`) and `gatefixer.RenderPrompt` (`internal/gatefixer/prompt.go:55`).

## Rejected

- guard-reach item 4, that no seat sets `ForkSubagents` or `Requires`: false. `burlerengine/review.go:87`, `burlerengine/fix.go:97`, `loomengine/plan.go:133` and `loomengine/discussiontable.go:158` set `ForkSubagents` on seats, so the unmet branch is reachable. The seat skip itself is raised as F4.
- guard-reach item 7, the `internal/orchengine` skip guarding no literal: D1 has the scan skip "the orch places D2 skips", so the plan follows the record.

Settled with no finding, per the focus:

- **Constraints.**
  - The `PATTERN-stencil-ownership` edit-directive change is already in its background (`pattern/PATTERN-stencil-ownership.md:83`).
  - The heddle background already names the hub reconcile (`pattern/PATTERN-heddle-instructions.md:25`).
  - This part adds no heddle target, no codex.yaml key and no hook run.
  - The `HEDDLE.md`, `PATTERN-config-layers` and `PATTERN-child-processes` constraints therefore fall outside cards 1–9.
- **Licensed scope.** Card 5's `NewWithSeams` serves D10's shared probe and hook runner, under `PATTERN-shuttle-provider-seam`. Card 6's two exported builder entry points serve D1's every-seat catalog.
- **Card 5's `Extends` on `pattern/PATTERN-shuttle-provider-seam.md`.** The card appends a sentence to the `codexengine.Seams` bullet and changes no existing text, so `Extends` is right.
- **Card 7's renderers.**
  - `Bouncer.RenderJudgePrompt(engine, round)` (`bouncer.go:1823`) and `Bouncer.RenderSeedPrompt(engine, focusPath)` (`bouncer.go:1762`) exist and take the engine.
  - `judgeCall` (`bouncer.go:1927`) and `runSeedSpawn` (`bouncer.go:1688`) render through them, so no split is needed.
  - The producer list matches every production `shuttleengine.Spec` literal that sets `Prompt`.
- **Card order.** `ListedSkills`, `Plugin`, `Hook`, `ErrHookUnrun`, `newParityFixture` and `seatTable` each come from an earlier card.
- **Overview.** The `## PATTERN changes` entry matches card 9's text and the declared entry.
