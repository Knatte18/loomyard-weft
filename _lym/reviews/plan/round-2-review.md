---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lym/plan/08-shuttle-manifest-verb.md:12
    summary: No card removes the now-stale internal/seatengine.Prompts entry from the test-only-code allowlist, so the PATTERN-test-only-code guard fails once the verb is wired
    fix: >-
      In card 8 add `- cmd/lym#testOnlyCodeAllowed` under `**Edit:**` and state in its Intent that the
      `internal/seatengine.Prompts` entry of `testOnlyCodeAllowed` in `cmd/lym/testonlycode_linux_test.go` is deleted,
      because `shuttlecli` now reaches `Prompts` through `shuttleparity.Report`, `seatsOf`, `seatengine.Specs`.
      Checked with `lym loom validate-plan --overlay` against the plan as it stands:
      the baseline run and the run with this one target added to card 8 report no blocking finding.
  - id: F2
    severity: BLOCKING
    class: design
    location: _lym/plan/06-seat-parity.md:87
    summary: Card 6 builds the burler fix table with nil values, so the fixer stencil has unfilled markers and Specs fails for every burler fix seat
    fix: >-
      In card 6's Intent change the `(*Engine).SeatTables` sentence so the fix table is built as `ProbeFix` builds it,
      `p.fixTable(opts, fixSeatValues(p, fixFilesIn(e.probeDirOf())), e.seatSkills(p))`, instead of with nil values.
      The fixer stencil `burler-template-fixer` fills `review_path`, `review_format_path` and `instruction_fix_path` from the seat's values alone,
      and `seatengine.Specs` renders every seat through `Prompts`, which refuses an unfilled top-level marker.
      The review table needs no change, since `probeTable` already passes `reviewSeatValues`, which fills every marker of the three reviewer stencils.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lym/plan/06-seat-parity.md:112
    summary: The parity fixture does not say where any seat skill comes from or how Codex finds it, so the skill-load channel can be empty or missing by construction
    fix: >-
      In card 6's `TestSeats_CodexManifestHoldsClaudes` paragraph, and by reference in card 7's `TestProducers_CodexManifestHoldsClaudes`, state three things.
      First, `newParityFixture` writes a hub `language.yaml` layer that lists at least one `<plugin>:<name>` skill, and the producer table's stand-in `LanguageSkills` are the same list,
      since seat and producer specs name skills only through language skills, so an empty list leaves the channel unexercised.
      Second, `PrepareEach` sets each spec's `Skills` with `cfg.Agents.Skills(spec.ConfigRole, spec.Skills)` when `ConfigRole` is non-empty, as `Runner.start` does before it calls `Prepare`,
      and the test collects the names it hands to `fx.addSkills` after that same call, so the role's own skills are compared too.
      Third, `addSkills` writes each name `<plugin>:<name>` as `<skill_roots dir>/<marketplace>/<plugin>/<version>/skills/<name>/SKILL.md`,
      the plugin-cache layout `skillPath` in `codexengine` resolves, and a name without a colon is a fixture error, since Codex lists no `skill-load` item for it whatever file exists.
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lym/plan/08-shuttle-manifest-verb.md:50
    summary: Card 8 makes the verb exit non-zero for an engine with an unmet requirement, which D10 and the every-seat decision do not license, and the overview says the verb only names it
    fix: >-
      In card 8's Intent replace the third bullet so an engine that cannot start the seat is named, with its unmet requirements from `shuttleparity.Unmet`,
      on a line of the `output.Ok` report and the exit stays zero when nothing is missing.
      Keep the non-zero exit for a missing compared item alone, as D10 says.
      Change the matching `TestManifestCmd` row to expect `ok` and exit zero with `codex` and the requirement named, and `TestReport`'s `Unmet` row stays as it is.
      Otherwise a Merriam or loom driver seat can never pass, since Codex meets no requirement, which the every-seat decision rules out.
  - id: F5
    severity: LOW
    class: consistency
    location: _lym/plan/06-seat-parity.md:77
    summary: Compare is said to gate the fork deny on Unmet for a ForkSubagents spec but its signature takes no spec
    fix: >-
      In card 6's Intent, and in the overview's decision on every seat, reword the gate so `Compare` drops the `guard` item keyed `shuttleengine.WebsterForkDenyKey`
      when the set's `codex` engine does not answer `RequirementForks` through `shuttleengine.RequirementSupporter`.
      It reads the engine's answer alone, not `Unmet` and no spec, and the Claude manifest holds the item only for a `ForkSubagents` spec.
      `TestCompare`'s fork-deny rows stay as written.
  - id: F6
    severity: LOW
    class: consistency
    location: _lym/plan/02-manifest-record.md:61
    summary: The way forward of each copyManifest warning names one cause though its error has several
    fix: >-
      In card 2's Intent, widen the way forward of the failed copy write to remove whatever blocks `<root>/manifests`,
      or make it a writable directory with free space, since the write also fails on a permission or a full disk.
      Widen the unreadable-record way forward to fix the engine's `shuttleengine.WriteManifest` call when the file does not decode,
      and to fix the run directory's `manifest.json` permissions when it cannot be read.
      Keep the two named rows of `TestRunner_Start_KeepsTheManifestCopy`.
---

### [BLOCKING] Stale test-only-code allowlist entry

**Location:** `_lym/plan/08-shuttle-manifest-verb.md` Edit list; `cmd/lym/testonlycode_linux_test.go` `testOnlyCodeAllowed`.

**Issue:** `PATTERN-test-only-code` is enforced by `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain` at tier 1.
Its allowlist holds `internal/seatengine.Prompts`, "oracle of the seat prompts for the tests of the packages that start seats", because no production code calls `Prompts` today.
Card 6 adds `seatengine.Specs`, which calls `Prompts`, and card 8 makes `shuttlecli` call `shuttleparity.Report`, which reaches `seatsOf`, the seat's `Specs` closure, `seatengine.Specs` and `Prompts`.
From card 8 on, a `main` package reaches `Prompts`, the allowlist entry matches no unreachable function, and `allowed.RequireNoStale` fails the guard.
No card edits `cmd/lym/testonlycode_linux_test.go`, and card 6 lists `internal/seatengine#Prompts` only under `Uses`.
The guard runs in the webster gate, so the plan lands red.

### [BLOCKING] The burler fix table is built with nil values

**Location:** `_lym/plan/06-seat-parity.md` line 87, the `(*Engine).SeatTables` sentence.

**Issue:** The card says `SeatTables` returns `probeTable(p, opts)` and `p.fixTable(opts, nil, e.seatSkills(p))`.
`fixTable` puts `values` on the seat, and `contracts/stencils/burler/burler-template-fixer.md` has the markers `review_path`, `review_format_path` and `instruction_fix_path`, none of them a reserved marker.
`seatengine.Specs` is `Prompts` plus `seatSpec`, and `Prompts` runs `composeSeatPrompt`, whose `stencil.FillWith` refuses an unfilled top-level marker.
So the solo and the fanned fix seat both fail in `Specs`, and `TestSeats_CodexManifestHoldsClaudes` fails by construction.
`ProbeFix` in `internal/burlerengine/fix.go` builds the same table with `fixSeatValues(&p, fixFilesIn(e.probeDirOf()))` for this reason.
The review side is fine: the three reviewer stencils, assisted and fanned ones included, need only `instruction_1_path`, `instruction_2_path`, `review_path` and `lens_report_path`,
and `reviewSeatValues` plus `assistSeat`'s own `lens_report_path` fill them.

### [MEDIUM] Skill fixture is underspecified

**Location:** `_lym/plan/06-seat-parity.md` lines 112-117; `_lym/plan/07-producer-table.md` line 61.

**Issue:** Claude's `skill-load` item exists for every skill the load message names, with no file needed.
Codex's exists only for a `<plugin>:<name>` skill whose SKILL.md sits at `<root>/<marketplace>/<plugin>/<version>/skills/<name>/SKILL.md` (`skillPath`, `skillFiles` in `codexengine/skillload.go`).
Card 6 says `addSkills` writes "a SKILL.md for each name under the fixture's `skill_roots` directory and the Claude config directory's skills directory", with no layout, so a flat `<root>/<name>/SKILL.md` leaves every skill missing by construction.
The card also never says where a seat's skills come from.
The seat tables name skills only through the language skills (`seatSkills`, `LanguageSkills` in each builder's input) and through `ConfigRole`, which `Runner.start` expands with `cfg.Agents.Skills` before `Prepare` and `PrepareEach` does not.
With no `language.yaml` layer in the fixture the collected list is empty, nothing is written, and dropping Codex's skill-load delivery fails no test, which breaks the acceptance criterion that dropping any one Codex delivery fails the test.
The role's own skills, which a real session loads, are never compared.

### [MEDIUM] The verb fails for an unmet requirement

**Location:** `_lym/plan/08-shuttle-manifest-verb.md` lines 50-51; `_lym/plan/00-overview.md` line 55.

**Issue:** D10 says the verb exits non-zero "and names each missing item when the Codex manifest lacks one", and its acceptance bullet repeats that.
The decision record's entry that every seat is compared says a seat Codex cannot start still has a manifest and is compared, and drops the skip.
The overview's decision says the verb "names the engine's unmet requirements", with no failure.
Card 8 adds an `output.Err` and a non-zero exit when an engine cannot start the seat.
Codex meets no requirement today, so `lym shuttle manifest` for Merriam's seat or the loom driver's seat can never pass, whatever its manifests hold.
The card introduces a failure mode the answer key does not license and the overview does not state.

### [LOW] Compare's fork-deny gate names a spec it is not given

**Location:** `_lym/plan/06-seat-parity.md` line 77; `_lym/plan/00-overview.md` line 51.

**Issue:** `Compare(engines, manifests)` takes no spec, yet card 6 drops the fork deny "while `Unmet` reports `RequirementForks` for the set's `codex` engine on a `ForkSubagents` spec", and `Unmet` takes a spec.
The gate can only be the engine's own answer; the sentence leaves the implementer to invent where the spec comes from.

### [LOW] copyManifest warnings name one cause each

**Location:** `_lym/plan/02-manifest-record.md` lines 61-63.

**Issue:** The copy write fails on a blocking file, a permission or a full disk, but the way forward names only what blocks `<root>/manifests`.
The unreadable-record warning comes from `state.ReadJSON`, which fails on a read error as well as on malformed JSON, but the way forward names only the engine's write.
Each way forward fits the one cause the card's own test row reaches and misleads for the others.

## Rejected

- Seat catalog reach of the assistant seat: `Build` calls `SeatTables` with an `Assist` stand-in, `probeTable` runs `assistSeat` and `withAssistant`, and `Engine.assistSeat` is in `Holders` for the scan.
  `withAssistant` holds a `[]seatengine.Seat` slice literal and an assignment of `chair.Stencil`, neither a `Seat` or `Table` literal, so it needs no entry.
  The only defect on that path is F2.
- Test-only reachability, cards 2, 6 and 8: the plan has `seatsOf`, not `Seats`.
  `Manifest.Omitted` is called by `WriteManifest`, which both engines' `Prepare` call; `MissingItems` by `Compare`; `Compare`, `PrepareEach`, `PromptItems`, `Unmet` and `LookupSeat` by `report`;
  `report` by `Report`, `seatsOf` by `report`, `UnrunHooks` and `ConfiguredSandbox` by the `shuttlecli` closure, all from `cmd/lym` once card 8 lands.
  `landingshed.FixReviewSeatTable` and `Engine.SeatTables` are reached through the `seatTables` entries' `Build` closures, a package variable initialised in `shuttleparity`'s `init`.
  `manifestEngine` and the fixture helpers are named only as test fakes; nothing places them in a production file.
  The one unreachable production function is `Prompts` through the stale allowlist, F1.
- Unmet-requirement skip: the plan carries none; card 6 says neither the parity tests nor `PrepareEach` skip a spec `Unmet` reports, and the decision record's every-seat entry records that.
  The remaining defect is the verb's exit, F4.
- Card 6 test coverage: `PrepareEach` on a failing `Prepare`, on an absent record, and `seatsOf` on a failing builder each have a named row in `TestCompare` in card 6.
- Card 1 test economy: `TestRenderer_EntryKeys` asserts `Slug` and `Engine`, which no existing test covers (`render_test.go` asserts texts only), and the plan adds no new Prepare test.
  The existing `TestPrepare_AppendsStandingPassages` tests keep their launch-text assertions and each planned change to them is a reader-type change or an item name.
- Card 2 launch statement: `start` documents that each failed attempt keeps its run directory, and `abandonStartup` keeps it, so a retried attempt keeps its copy under its own run id, as the card says.
  Every failure before `saveRunState` removes the run directory and writes no copy.
- Card 2 scans: `findRunByStrand`, `sweepOrphans`, `collectAttachCandidates`, `EnginesRun`, `ReadSessionStates`, `ReadWaitMarkers`, `carriedInstructions`, `StopStepRuns` and the activity readers skip a directory with no `run.json`, as the card says.
- Producer renderers named in card 7 all exist and are exported, `shedarbiter.Prompt` included, so card 7 needs no production change.
