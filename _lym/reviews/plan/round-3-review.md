---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lym/plan/06-seat-parity.md
    summary: >-
      Card 6 creates the new test package internal/shuttleparity and no card creates its TestMain,
      so PATTERN-test-isolation's tmuxkit.Main guard fails for the package
    fix: >-
      In 06-seat-parity.md add to the Create group the line
      - `plan:internal/shuttleparity#TestMain` -> `func TestMain(m *testing.M)`
      and add to the Intent a sentence that TestMain mirrors internal/seatengine/testmain_test.go:
      it calls gitkit.HermeticGitEnv() and then os.Exit(tmuxkit.Main(m)), in a _test.go file of the package.
      Checked with lym loom validate-plan --overlay together with F2 and F4: no blocking finding beyond the baseline.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lym/plan/05-engines-with-seams.md, _lym/plan/06-seat-parity.md, _lym/plan/08-shuttle-manifest-verb.md
    summary: >-
      Cards 5 and 6 add production functions no main package reaches until card 8,
      and no card admits them in testOnlyCodeAllowed, so a batch gate that ends before card 8 fails the test-only-code guard
    fix: >-
      Follow the precedent of the internal/runfsm.Order entry in cmd/lym/testonlycode_linux_test.go
      (no caller until a later card, which removes the entry).
      Card 5 - add `cmd/lym#testOnlyCodeAllowed` to its Edit group and an Intent sentence adding the entry
      internal/shuttleengines.NewWithSeams (Why - no caller until lym shuttle manifest lands, card 8 removes this entry).
      Card 6 - add an Edit group holding `cmd/lym#testOnlyCodeAllowed`, an ImpactSummary line
      (an Edit group requires one; for example - the allowlist gains interim entries for the production functions no main reaches until card 8,
      and card 8 removes them), and an Intent sentence adding the entries
      internal/seatengine.Specs, internal/landingshed.FixReviewSeatTable and internal/burlerengine.Engine.SeatTables with the same reason.
      Card 8 - extend its Intent sentence on the allowlist so it deletes those four entries together with internal/seatengine.Prompts,
      and mention them in its ImpactSummary.
      Functions of internal/shuttleparity itself need no entry, since no main or test-support package imports it before card 8.
      Checked with lym loom validate-plan --overlay together with F1 and F4: no blocking finding beyond the baseline.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lym/plan/06-seat-parity.md
    summary: >-
      Card 6 Seat.Specs swaps only the seat's Engine and keeps its Model, Effort and Version,
      so Codex's validateChoice refuses a Claude effort or version pin and lym shuttle manifest errors on a hub whose seat models carry params
    fix: >-
      In 06-seat-parity.md, in the Intent paragraph on seatsOf and Seat.Specs, state that the spec Specs returns carries
      ModelChoice{Engine: engine} alone: Model, Effort and Version are cleared,
      since the manifest compares what the session is given and not which model runs it, and each engine validates its own model vocabulary.
      Add one row to the existing seatsOf row of TestCompare: a fixture table whose seat ModelChoice carries Model, Effort and Version
      gives, for each engine, a spec whose ModelChoice is that engine alone.
      No target is added or moved.
  - id: F4
    severity: LOW
    class: consistency
    location: _lym/plan/05-engines-with-seams.md
    summary: Card 5 changes the parameters of build but lists its Edit glyph without the re-sign arrow
    fix: >-
      In 05-engines-with-seams.md, in the Edit group, replace the line - `internal/shuttleengines#build`
      with - `internal/shuttleengines#build` -> `func build(cfg shuttleengine.Config, dirs configengine.Dirs, constructors map[string]constructor, seams LaunchSeams) (shuttleengine.EngineSet, error)`.
      Checked with lym loom validate-plan --overlay together with F1 and F2: no blocking finding beyond the baseline.
  - id: F5
    severity: LOW
    class: consistency
    location: _lym/plan/06-seat-parity.md
    summary: >-
      Cards 6 to 8 name the engines claude and codex in Compare, the manifest map keys and the report without saying where the names come from,
      against PATTERN-code-conventions' one named constant rule
    fix: >-
      In 06-seat-parity.md state in the Intent that internal/shuttleparity declares the reference engine name and the compared engine name
      once each as named constants, and that Compare, PrepareEach's manifest keys, Report and the tests use them
      (the engines' own constants are unexported and the engine name is a registry key, which PATTERN-shuttle-provider-seam allows outside the engine directories).
      No target is added or moved.
---

### [BLOCKING] No TestMain for the new internal/shuttleparity test package

**Location:** `_lym/plan/06-seat-parity.md` (Create group and Tests), `_lym/plan/07-producer-table.md`, `_lym/plan/08-shuttle-manifest-verb.md`

**Issue:** Card 6 creates package `internal/shuttleparity` with `TestCompare` and `TestSeats_CodexManifestHoldsClaudes`; cards 7 and 8 add more tests to it.
`PATTERN-test-isolation` requires every test package to run through `tmuxkit.Main`.
`cmd/lym/tmuxisolation_test.go` (`tmuxIsolationFailures`) fails with "no TestMain compiles under tags ... on <platform>" for any directory holding a compiled test file and no `TestMain`.
No card under `_lym/plan/` lists a `TestMain` or a `testmain_test.go`, so the first batch gate that runs `cmd/lym`'s guards after card 6 fails.
A `Create` glyph for `TestMain` is the admissible fix; the overlay run reported no blocking finding for it.

### [MEDIUM] Interim reachability of cards 5 and 6 is not admitted in `testOnlyCodeAllowed`

**Location:** `_lym/plan/05-engines-with-seams.md`, `_lym/plan/06-seat-parity.md`, `_lym/plan/08-shuttle-manifest-verb.md`

**Issue:** `PATTERN-test-only-code` requires every production function to be reachable from a main package;
`cmd/lym/testonlycode_linux_test.go` is `//lym:guard` at tier 1, so a narrowed batch gate runs it by name whatever the impacted set.
`shuttleengines.NewWithSeams` (card 5) is called by no main until card 8's `shuttlecli` closure.
`seatengine.Specs`, `landingshed.FixReviewSeatTable` and `burlerengine.Engine.SeatTables` (card 6) are called only by `internal/shuttleparity`, which no main imports until card 8, so the analysis never loads it and its callees read as unreachable.
Card 8 handles only the end state, deleting the `internal/seatengine.Prompts` entry.
The file already carries the pattern for the interim state: `internal/runfsm.Order` is admitted with a reason saying a later card removes the entry.
The plan states "a valid plan is executable strictly in card-number order", so each prefix of the card order must pass the guards the batch gate runs.
Whether a batch boundary falls between cards 5 or 6 and card 8 is Webster's choice, so the plan must not rely on it.

### [MEDIUM] `Seat.Specs` keeps the seat's Claude-side model parameters when it swaps the engine

**Location:** `_lym/plan/06-seat-parity.md` (Intent paragraph on `seatsOf` and `Seat.Specs`)

**Issue:** `seatengine.seatSpec` copies `seat.ModelChoice` whole into the spec, and card 6 only sets its `Engine`.
`codexengine.validateChoice` refuses a non-empty `Version` and an effort outside `codex.yaml`'s `efforts` (the template lists low, medium, high), and `claudeengine.Prepare` runs `validateEffort(spec.Effort)` and `resolveModelID(spec.Model, spec.Version)`.
A hub whose seat model-spec names `[effort=xhigh]`, a `v=` pin, or a Codex model with a Codex-only effort makes one engine's `Prepare` refuse the spec, so `PrepareEach` returns an error and `lym shuttle manifest` exits non-zero for a reason that has nothing to do with a missing item.
That is the verb's main use: a bench seat configured for Codex, whose Claude counterpart must be printed beside it.
The fixture's template-default models carry no params, so `TestSeats_CodexManifestHoldsClaudes` passes and hides the defect.
Producers in card 7 set no `ModelChoice` and are unaffected.

### [LOW] `build` loses its signature without the re-sign arrow

**Location:** `_lym/plan/05-engines-with-seams.md`

**Issue:** The Intent says "`build` takes the seams and hands them to each constructor", which adds a parameter to `internal/shuttleengines#build`; the Edit glyph carries no arrow.
`constructor` carries one and `New`, `KeepWarm` and the other callers keep their signatures, so only `build` is affected.

### [LOW] Engine names appear in cards 6 to 8 with no stated source

**Location:** `_lym/plan/06-seat-parity.md`

**Issue:** `Compare` is "`MissingItems(claude, codex)`" and gates on "the set's `codex` engine"; `PrepareEach` returns manifests "by engine name"; `Report` and the tests repeat both names.
`PATTERN-code-conventions` says a name that can go stale is one named constant and a change never adds another literal.
The engines' `engineName` constants are unexported, so an implementer reading the card has only the literals to reach for.
The engine name is a registry key that `PATTERN-shuttle-provider-seam` allows outside the engine directories, so one pair of constants in `internal/shuttleparity` satisfies both entries.

## Focus verdicts

- Cards 6 and 8, `testOnlyCodeAllowed`: once `shuttlecli` calls `shuttleparity.Report`, `internal/seatengine.Prompts` becomes reachable through `Report`, `seatsOf` and `seatengine.Specs`, so its entry is stale and card 8 deletes it; card 8 lists `cmd/lym#testOnlyCodeAllowed` under `Edit`, so a card does edit the file.
  No other entry changes status: `claudeengine.New` is not reached by the registry (`NewFromConfig` is), and the other entries name functions the plan does not touch.
  The end state is correct; the interim state is F2.
- Card 6 `(*Engine).SeatTables`: the fixer stencil `burler-template-fixer` carries `review_path`, `review_format_path` and `instruction_fix_path` besides the reserved markers, and `fixSeatValues(p, fixFilesIn(e.probeDirOf()))` is exactly what `ProbeFix` passes, so the fix table fills every top-level marker.
  The review table's reviewer, assisted-reviewer and assistant stencils fill from `reviewSeatValues` and `assistSeat`'s own `lens_report_path`.
  Not a defect.
- Cards 6 and 7 fixtures: `PrepareEach` expands `cfg.Agents.Skills(spec.ConfigRole, spec.Skills)` as `Runner.start` does, the template's role lists are all `<plugin>:<name>` entries, and `addSkills` writes `<root>/<marketplace>/<plugin>/<version>/skills/<name>/SKILL.md`, the layout `skillPath` and `skillFiles` resolve.
  The roles with an empty template list (arbiter, driver, tackler) leave their skill channels empty, which is the real behavior.
  Not a defect.
- Card 8 exit codes: only a missing compared item or an error from `Report` exits non-zero, and a seat Codex cannot start passes when nothing is missing.
  That follows D10 and the "every seat is compared" decision, since `Prepare` refuses no requirement.
  Not a defect; the one gap is the model parameters that make an error from `Report` likely for a real hub (F3).
- Card 6 `Compare`: the fork-deny gate needs the `codex` engine's `RequirementSupporter` answer, which `engines.Lookup` gives, and Claude's manifest holds the item only for a `ForkSubagents` spec, so every condition is computable from `engines` and `manifests`.
  Only the engine names need a stated source (F5).
- Card 2 `copyManifest`: `state.ReadJSON` fails as `ErrDecode`, `ErrRead` or a lock error, and the warning's two clauses (re-write the manifest, fix permissions) cover them; the write warning's "remove whatever blocks `<root>/manifests`, or make it a writable directory with free space" covers a blocked path, permissions and a full disk.
  The table rows reach the decode and the blocked-directory cases, and one message holds both clauses of the unreadable-record warning.
  Not a defect.

## Rejected

- Card 7's producer list: every production `shuttleengine.Spec` literal that sets `PromptEngine` is either in the list or a `gatefixer.Spec`, and the bouncer's `judgeCall` and `runSeedSpawn` already render through `RenderJudgePrompt` and `RenderSeedPrompt`, so no split is owed.
- Card 6's seat catalog against the production `seatengine.Seat` literals: every builder holding one is listed, bar the `MultiLLM` rows the card bounds out.
- Card 2's claim that the other run-root scans skip a directory with no `run.json`: they do skip it; `loadRunState` leaves a `run.json.lock` file inside `manifests`, which is harmless and not a finding.
- Card 6 hosting `PrepareEach`, `Unmet` and `seatsOf` rows in `TestCompare`: a naming nit only, no behavior is double-covered.
