---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/only.go#markedScope
    summary: "An upkeep call that runs none of its marked tests exits 0 and reads as a clean upkeep. On this hub, which sets no gates.upkeep and so takes the template's tiers [0], every upkeep test is tier 1, so the dead-code guard and both .NET tests now run at no gate at all."
    fix: "In markedScope, refuse the call with a non-zero exit when a marked test sits in a tier the key's listed tiers neither are nor cover. Name the tests and the way forward, setting gate.gates.upkeep to list that tier with `lym config gate` from the prime, so the board-tidy agent sees a failure and files a bug note. In runTest (internal/gatecli/run.go), after planTierRuns, make a marked-scope call whose guard step holds no test exit non-zero with a line naming why: no go.mod in the scanned directory, or no test marked for the key. Add TestGateTest_Scopes rows for a marked test of an unlisted tier and for a marked key with no marked test, each asserting the refusal."
  - id: F2
    severity: BLOCKING
    class: consistency
    origin: handler
    location: internal/gatecli/doc.go:266
    summary: "PATTERN-config-key-names is broken: new prose names config keys in short form. internal/gatecli/doc.go:266 names the key `gates`, and internal/gateslot/template.yaml:20 writes `gates.upkeep`."
    fix: "In internal/gatecli/doc.go:266 write `gate.gates` in place of `gates`. In internal/gateslot/template.yaml:20 write `gate.gates.upkeep` in place of `gates.upkeep`, or name the full command, such as `lym config gate --set 'gates.upkeep={tiers: [0, 1], scope: marked}'`, where the short path is the command's argument rather than a key named in prose."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/tiers.go#scopeInvocations
    summary: "A key of scope marked runs its listed tiers whole under --no-carry, or when a listed tier is set to carry false. planTierRuns sets tier.NoCarry from request.noCarry, and scopeInvocations returns runsWhole for tier.NoCarry before it reaches the empty-modules case."
    fix: "In scopeInvocations, add a first case `sc.form == scopeMarked` that returns nil, false, so a marked scope's listed tiers plan no invocation under any flag or tier setting. Add a TestPlanTierRuns row with scope{form: scopeMarked} and request.noCarry set, asserting the listed tier has no invocation."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/beamcli/dobbydeps.go:409
    summary: "The prime's UpkeepCall renders the upkeep call whatever the scoper. markedScope always refuses that call under the none scoper (the template default) and under dotnet, so every non-Go hub's board-tidy agent runs a refused call at each tidy."
    fix: "In the UpkeepCall closure in hubDobbyDepsStarting, return an empty call with a nil error when cfg.Scoper is not go, so TidyPrompt takes the no-upkeep step. Cover it with the wiring test asked for in F7."
  - id: F5
    severity: MEDIUM
    class: scope
    origin: lens:code-scope
    location: cmd/lym/removedverbs_test.go:40
    summary: "Two whole-repository scans in cmd/lym carry no marker and so get no place: TestRemovedVerbs_NamedNowhere and TestAgentMessaging_NoEngineTool (cmd/lym/agentmessaging_test.go:95). They still run in every task gate that impacts cmd/lym, though the board entry asks for every whole-repo guard in cmd/lym to be named a place."
    fix: "Add `//lym:only parent-align finalize` as a directive line directly above `func TestRemovedVerbs_NamedNowhere` in cmd/lym/removedverbs_test.go and above `func TestAgentMessaging_NoEngineTool` in cmd/lym/agentmessaging_test.go. Then name in pattern/PATTERN-test-speed.md that an unmarked whole-repository scan of cmd/lym counts too. Alternatively, mark each one `//lym:guard` and give it its place the same way."
  - id: F6
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/landing/summary.md#Merge danger
    summary: "The change description calls the door two-way, but the diff changes two things another binary reads. The green record gains `held`, which an older binary ignores, so it can carry a pass over held tests. The gate config gains scope marked, which an older binary's decodeGates refuses, so its every gate call fails once a hub sets gates.upkeep as the description advises. The blast radius names only the first."
    fix: "In _lym/landing/summary.md under Merge danger, set the door to one-way. Name both effects: an older binary trusting a `held` entry, and an older binary refusing the gate config once it holds a key of scope marked. State the recovery for each: remove the green records under the wait directory, and drop the marked key from the hub's gate config."
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/gatecli/scopes_integration_test.go:575
    summary: "No test pins the behaviours that move the guards off the task gates or keep the carry honest. Nothing covers the guard step leaving out a held //lym:guard test (withoutHeld; deleting run.go:502 passes every test). Nothing covers recordPass or recordInertCarried writing Held into an entry, nor the prime-only UpkeepCall wiring and daemon.upkeepCall's fallback."
    fix: "In the TestGateTest_Scopes subtest at scopes_integration_test.go:575, give the fixture a second test marked both `//lym:guard` and `//lym:only finalize`, outside the impacted set. Assert that a computed call (no --gate key naming it) leaves it out of the guard step's -run filter. In internal/gatecli, add an assertion that the green entry a held run writes carries Held equal to the held names. Read the record the integration run writes; do not build it by hand. In internal/dobbyengine, add a test that spawnTidy with a nil UpkeepCall, and with one returning an error, renders the no-upkeep step. In internal/beamcli, add a row that the prime's deps set UpkeepCall and a beam's do not."
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/only.go#withOnlyTests
    summary: "Several added branches have no test: a failed go list (warn and hold nothing, or refuse a marked key); markedScope's refusal under a scoper other than go; greenRecord.with keeping entries with different Held apart; skippedInside dropping a guard whose tier a listed tier covers; the -C scan directory of explicit targets; explicit targets replacing a marked key."
    fix: "Add a TestPlanTierRuns or unit row for each branch: a skippedInside call where tier 1 covers tier 0, dropping a tier-0 inside guard; a greenRecord.with call over two entries differing only in Held, asserting both stay. In TestGateTest_Scopes, add a marked-key call under the none scoper asserting the refusal text, and a `--gate upkeep ./heldpkg` call asserting the targets scope runs."
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/gatecli/run.go:502
    summary: "The guard step drops held tests even when the words after -- carry a filter flag. gatecli/doc.go and the change description both say such a call holds nothing back."
    fix: "In runTest, apply `sc.guards = withoutHeld(sc.guards, request.held)` only when request.args carry no filter flag (`!slices.ContainsFunc(request.args, isFilterFlag)`). Or, in withOnlyTests, leave request.held empty in that case, so the listed runs and the guard step agree with the documented contract."
  - id: F10
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/only.go#withOnlyTests
    summary: "Every --gate call under the go scoper now runs an extra go list through modulePackageDirs before computeScope. No context bounds it and no preTierExpiry check follows, so a stalled listing hangs the call past the pre-tier ceiling."
    fix: "Pass preCtx into withOnlyTests and on into a context-taking variant of impactset.ModuleOnlyTests and modulePackageDirs, using exec.CommandContext. Call preTierExpiry with \"scanning the //lym:only tests\" right after withOnlyTests in runTest."
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/dobbyengine/doc.go:204
    summary: "The new bullet says the agent runs the upkeep checks first, but the stencil makes them step 3, after the tidy steps. The bullet also sits above the line about board_tidy_timeout_min, which now reads as part of it."
    fix: "In internal/dobbyengine/doc.go, change `runs the upkeep checks first` to `also runs the upkeep checks before its report`. Move the `Past board_tidy_timeout_min ...` line back under the spawn bullet, above the new upkeep bullet."
---

### [BLOCKING] An upkeep run that runs none of its tests passes silently, and on this hub it runs none

**Location:** `internal/gatecli/only.go#markedScope`

**Issue:** `markedScope` prints a stderr line and `continue`s for each marked tier that the key's listed tiers neither are nor cover.
`runTest` only logs at Info when the guard step is left with nothing to run, and the call exits 0.
This hub's gate config (`_bolt/_lym/config/gate.yaml`) sets no `upkeep` key, so the key takes the template's `{tiers: [0], scope: marked}`.
`TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain`, `TestGateRedFirst_Dotnet` and `TestGateTest_DotnetFixture` all call `tiergate.Require(t, 1)`.
Every other key now holds them back, so after landing the dead-code guard and both .NET tests run at no gate on this hub.
The board-tidy agent sees a passing call and files nothing.
The board entry asks for these checks to run in the upkeep run and to turn into notes.
The change description admits the config gap; nothing in the diff closes it or makes it visible as a failure.

### [BLOCKING] New prose names config keys in short form

**Location:** `internal/gatecli/doc.go:266`

**Issue:** `PATTERN-config-key-names` asks for a config key named in a comment, doc or stencil to be written in full, as `<module>.<dotted path>`.
The new Held-tests paragraph writes "a key `gates` does not hold".
The new template comment at `internal/gateslot/template.yaml:20` writes "sets gates.upkeep={...}", while line 17 of the same file writes `gate.gates`.
`pattern/PATTERN-test-speed.md` in the same diff writes `gate.gates` in full.

### [MEDIUM] A marked key runs its listed tiers whole under --no-carry or carry: false

**Location:** `internal/gatecli/tiers.go#scopeInvocations`

**Issue:** `planTierRuns` sets `tier.NoCarry = tier.NoCarry || request.noCarry`.
`scopeInvocations` checks `case sc.whole, tier.NoCarry: return runsWhole` before `case len(sc.modules) == 0`.
So `lym gate test --gate upkeep --no-carry` runs tier0 over its whole `all_targets`, minus the held tests, besides the marked tests.
The focus asks that the key run exactly its marked tests under any flag combination.

### [MEDIUM] Non-Go hubs get an upkeep call that is always refused

**Location:** `internal/beamcli/dobbydeps.go:409`

**Issue:** The closure renders `cfg.AgentGateCall(gateslot.UpkeepGateKey)` whatever the scoper.
The template's `scoper` is `none`, and `markedScope` refuses any scoper but `go`.
Every non-Go hub's prime board-tidy agent therefore runs a refused call at each tidy, and the stencil tells it to file notes only for failing tests.

### [MEDIUM] Two whole-repository scans in cmd/lym get no place

**Location:** `cmd/lym/removedverbs_test.go:40`

**Issue:** `TestRemovedVerbs_NamedNowhere` and `TestAgentMessaging_NoEngineTool` walk the whole repository through `scankit.Walk` but carry no `//lym:guard`.
So the diff's sweep, which marked only `//lym:guard` tests, left them unmarked, and they still run in every task gate whose scope holds `cmd/lym`.
The board entry asks for every whole-repo guard in `cmd/lym` to be found and named a place.

### [MEDIUM] The Merge danger door is called two-way while the diff changes formats another binary reads

**Location:** `_lym/landing/summary.md#Merge danger`

**Issue:** The rubric flags a `two-way` call when the diff changes a record or config format another binary reads.
The description itself says an older binary ignores `held` and can carry a pass that skipped the held tests.
It does not name the second effect: `decodeGates` in an older binary refuses `scope: marked`.
So once a hub sets `gates.upkeep`, as the description advises, every gate call of an older binary fails at config load.

### [MEDIUM] The behaviours that move guards off the task gates have no test

**Location:** `internal/gatecli/scopes_integration_test.go:575`

**Issue:** The only fixture with a `//lym:only` marker carries no `//lym:guard`, and its `finalize` row runs scope `all`, which plans no guard step.
So no row fails if `withoutHeld` stops dropping held guards, and that drop is what takes every `cmd/lym` guard off the task gates.
The `held` rows of `TestGreenRecord` build entries by hand, so no row fails if `recordPass` or `recordInertCarried` stops writing `Held`.
Without that field, a webster pass would carry Finalize over the landing checks.
`grep UpkeepCall` over the tests finds only `TestTidyPrompt`, so the prime-only wiring and the render-error fallback are unpinned.

### [LOW] Several added branches have no test

**Location:** `internal/gatecli/only.go#withOnlyTests`

**Issue:** No test covers:
- a failed `go list`, warning and holding nothing, or refusing a marked key;
- `markedScope`'s refusal under a scoper other than go;
- `greenRecord.with` keeping entries with different `Held` apart;
- `skippedInside`'s covers branch;
- the `-C` scan directory;
- explicit targets beside a marked key.

### [LOW] The guard step drops held tests under a filter flag, against the documented contract

**Location:** `internal/gatecli/run.go:502`

**Issue:** `planTierRuns` adds no skip when the words after `--` carry a filter flag, as `gatecli/doc.go` says.
But `withOnlyTests` still fills `request.held`, and `withoutHeld` still drops those tests from the guard step.
So `--gate card --card 1 -- -run X` leaves out held guards its guard step used to run.

### [LOW] The added go list on every --gate call has no time bound

**Location:** `internal/gatecli/only.go#withOnlyTests`

**Issue:** `modulePackageDirs` runs `go list` through `exec.Command` with no context, and `withOnlyTests` is handed neither `preCtx` nor `ctx`.
It now runs on every `--gate` call under the go scoper, scope-all landing keys included, before `computeScope` and with no `preTierExpiry` check after it.

### [LOW] The dobby doc misplaces the upkeep step and the timeout line

**Location:** `internal/dobbyengine/doc.go:204`

**Issue:** The new bullet says the agent runs the upkeep checks "first", but the stencil makes them step 3, after the tidy steps.
The bullet sits between the spawn bullet and its timeout line, so the timeout line now reads as part of the upkeep bullet.

## Rejected

- Deleting `cmd/lym/testsupportdirs_linux_test.go` and `internal/planparser/latercaller_test.go` is no loosened check: the behaviour they pinned, the `create-caller-later` check and its directory list, is itself removed, and no reference to it is left.
- Moving the landing checks off the `webster` and `adopt` keys is no gap: the board entry asks for them to run once at landing, and Parent-Align and Finalize both run them over the whole tree. Finalize carries Parent-Align's pass because the two keys hold back the same tests.
- `goReadGates` leaving out `upkeep` is no defect: no Go verify site reads that key.
- `TestCSharpFixtures_CompileWithDotnet` staying at the task gates is not an unplaced guard: it carries `//lym:watch *.cs`, and the change description's rule keeps watched guards at the task gates.
- Matching held and marked tests by bare name is a bound, not a defect today: no held name occurs twice in the repository, and the watch skip already documents the same bound.

## Focus departures

- "Landing coverage: trace every route by which work reaches main or a parent (parent-align, finalize, adopt, beam, PR and darn paths, ...)": I traced the gate keys and the green-record carry between the webster, Parent-Align and Finalize keys. I did not trace the beam pull-request and darn routes in landingshed's code. Instead I relied on the `PATTERN-landing` rule that a landing happens only in a run's Finalize.
- "Board-tidy upkeep step: settle whether the agent can finish the upkeep call and file notes within its timeout": the board-tidy timeout defaults to 30 minutes (`internal/dobbyengine/template.yaml:3`), and no shell-guard row denies the call. I did not measure the upkeep run's length, so whether it fits rests on the reported figures: a call-graph test peaking at about 2 GB, and .NET tests of about 85 s together.
