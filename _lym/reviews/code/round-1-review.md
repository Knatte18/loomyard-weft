---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: internal/gatecli/scope.go#scanGuards
    summary: "The widened card scope puts cmd/lym, which depends on nearly every internal package, inside almost every card's scope. scanGuards then drops every guard of an inside package from the guard step whatever its tier, and the card key lists tier0 alone, so cmd/lym's tier1 guards (TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain among them) and the tier1 guards of other dependents now run nowhere at the card gate."
    fix: "In scanGuards (internal/gatecli/scope.go), keep a guard test of an inside package in the guard step unless the call's listed tiers run its tier: count it as inside only when its GuardTest.Tier is a kept listed tier or a tier a kept listed tier covers. Pass that tier set in from the call (computeScope can store it on the scope, or scanGuards can take cfg and request and use keptTiers). Add a TestGateTest_Scopes row in internal/gatecli/scopes_integration_test.go: a package that imports ./lib and holds a //lym:guard test of a tier the call does not list, asserting that the --card call's guard step runs it. Update the change description's sentence that the guard step works from the widened set so it states this rule."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/gatecli/scope.go#cardDependents
    summary: "When go list cannot list the module, cardDependents only calls logger.Warn and the card gate runs, and can pass, over the cards' packages alone. Nothing tells the caller, while the diff scope names its fallback on stderr and --main-packages refuses on the same failure. Since listPackages runs without -e, one broken package anywhere in the module drops every dependent silently."
    fix: "Give cardDependents the request's stderr writer and print a line there naming the module and the cause, such as \"gate test: the card scope holds the cards' packages alone: cannot list the packages that depend on them in <module>: <cause>\", keeping the logger.Warn. Update the cardDependents doc comment and internal/gatecli/doc.go line 142 to say the narrowing is named on stderr. In TestCardDependents' failure row (internal/gatecli/carddependents_integration_test.go), assert that line on the writer as well as the returned slice."
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: contracts/stencils/webster/webster-body-implementer.md:73
    summary: "The stencil still names the guard step as the step that fails on a cmd/lym pinned set, and sanctions changing only a file the guard step's failure names. With cmd/lym now inside the card scope, its tier0 guards run in the listed tier0 run, so such a failure falls outside the deviation the stencil sanctions."
    fix: "In webster-body-implementer.md, extend the sanctioned deviation to a //lym:guard test that fails in a listed tier's run over a package outside the card's targets: the sentence at line 73 says the gate call's runs, the listed tiers over the dependent packages as well as the guard step, can fail on a file the card does not list, such as the pinned set in a cmd/lym test. Change the later sentence that says \"Change only a file the guard step's failure names\" to name a failure of either."
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/gatecli/doc.go:211
    summary: "The vet-step section still says a tag brings a file into a card package and that the run covers the card's packages, but the vet runs and tagTiers now work over the widened set, as the changed row expecting vet -tags tier1 ./lib ./user shows."
    fix: "In internal/gatecli/doc.go's # Vet step section, replace \"a card package\" with \"a package of the card scope\" and \"The run covers the card's packages, once per module\" with \"The run covers the card scope's packages, once per module\"."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/testmap_integration_test.go#TestGateTestMapInATaskWorktreeKeepsItsOwnMaps
    summary: "The change description says --map --card now maps the widened set, but no test has a card scope with a dependent package; the only --map --card fixture is a single-package module."
    fix: "In the --map --card test, add a second package to the fixture module that imports solo, and assert that the call's built or packages output names both solo and the importer."
  - id: F6
    severity: NIT
    class: consistency
    origin: lens:code-tests
    location: internal/gatecli/carddependents_integration_test.go:28
    summary: "The success row of TestCardDependents asserts only DependentPackages' result passed through, which TestDependentPackages' first row (internal/impactset/dependents_integration_test.go) and the --card row of TestGateTest_Scopes already pin over the same lib and user shape."
    fix: "Delete the row \"a module that lists adds the packages that import the card's packages\" from TestCardDependents, keeping the failure row."
  - id: F7
    severity: NIT
    class: design
    origin: lens:code-correctness
    location: internal/impactset/dependents.go:38
    summary: "dependentsFor looks a seed up by path.Clean(arg) but de-duplicates against the raw arg, so an unclean seed such as ./lib/ is listed twice, once raw and once as ./lib."
    fix: "In dependentsFor, build the result from the cleaned spelling (\"./\"+path.Clean(arg), or \".\" for the root) so the seed lookup and the de-duplication compare the same string, or drop the path.Clean and state in the DependentPackages doc that the arguments are clean."
---

### [BLOCKING] Widened card scope stops the tier1 guards of cmd/lym and other dependents from running at the card gate

**Location:** `internal/gatecli/scope.go#scanGuards`, reached through the `cardDependents` call at `internal/gatecli/scope.go:198`.

**Issue:** `scanGuards` builds `inside` from every target of the scope and keeps a guard test only when its directory is outside it (`scope.go:582-590`), whatever the guard's tier.
Before this diff a card scope held only the card's own packages, so `cmd/lym`'s guards stayed outside and ran in the guard step under their own tier.
`go list -deps ./cmd/lym` lists 188 of the module's internal packages, `planparser`, `shedrecipe`, `impactset` and `gatecli` among them.
So `DependentPackages` now adds `./cmd/lym` for almost any card, and `cmd/lym` is inside.
The `card` key is `{tiers: [0], ...}` (`internal/gateslot/template.yaml:35`), and the hub's `gate.yaml` does not override it.
`cmd/lym` holds three `tier1` guard tests, among them `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain` (`cmd/lym/testonlycode_linux_test.go:79-81`).
The listed tier0 run skips them through `tiergate.Require(t, 1)`, and the guard step no longer lists them, so the card gate never runs them.
The same holds for the tier1 guards of `internal/gatecli`, `internal/impactset` and `internal/loomshed` whenever they join the scope as dependents.

This breaks `PATTERN-test-speed`'s rule that a narrowed gate's guard step runs every marked test by name under its file's tier, whatever the impacted set.
It also falsifies `internal/gatecli/doc.go`'s guard-step claim that a narrower gate keeps full guard coverage.
It even switches off at the card gate the guard that issue #262 reports, which the change description says the diff leaves as strict as it was.
The change description's sentence that the guard step works from the widened set states the change without this consequence.
No test catches it, because the `TestGateTest_Scopes` fixture's guard package imports no card package.

### [MEDIUM] A failed dependents listing narrows the card gate without telling the caller

**Location:** `internal/gatecli/scope.go#cardDependents`

**Issue:** On a `DependentPackages` error, `cardDependents` logs at Warn and returns `module.Packages` (`scope.go:286-293`).
The call then runs, and can pass, over a narrower scope than the gate's documentation and the `--card` help now promise, and nothing reaches `request.stderr`.
The diff scope prints its fallback there (`scope.go:256`), and `--main-packages` refuses on the same `go list` failure (`scope.go:203-204`).
`listPackages` runs `go list` without `-e` (`internal/impactset/derive.go:60`), so a load error in any package of the module drops every dependent.
A broken package outside the card's packages is then compiled by no listed tier, and the card gate can go green.

### [MEDIUM] Implementer stencil's guard deviation no longer covers a cmd/lym guard that fails in the tier run

**Location:** `contracts/stencils/webster/webster-body-implementer.md:73`

**Issue:** The diff edits this sentence to add the dependents, but keeps the guard step as the step that fails on the pinned set in a `cmd/lym` test.
It also keeps the rule that the implementer changes only a file the guard step's failure names.
With `cmd/lym` inside the card scope, its tier0 guards now run in the listed tier0 run over `./cmd/lym`, not in the guard step.
A pinned-set failure there is outside the sanctioned deviation, so the stencil tells the implementer it may not fix the very failure it names as expected.

### [LOW] Vet-step section of the gatecli doc still speaks of the card's packages

**Location:** `internal/gatecli/doc.go:211`

**Issue:** The `# Vet step` section says a tier gets a vet run when its tag brings a file into a card package, and that the run covers the card's packages.
`tagTiers` and `vetStepRun` now take the widened modules, as the changed row `[]string{"vet", "-tags", "tier1", "./lib", "./user"}` in `scopes_integration_test.go` shows.
The diff updated the scope paragraph of the same doc and left this section unchanged.

### [LOW] No test covers --map --card over a widened scope

**Location:** `internal/gatecli/testmap_integration_test.go#TestGateTestMapInATaskWorktreeKeepsItsOwnMaps`

**Issue:** `candidatePackages` (`internal/gatecli/testmap.go:426-437`) reads `share.modules`, which now hold the dependents.
The change description claims the widening covers `--map --card`, but the one `--map --card` fixture is a module with a single package, so no row would fail if the map call dropped the dependents.

### [NIT] TestCardDependents' success row repeats what TestDependentPackages pins

**Location:** `internal/gatecli/carddependents_integration_test.go:28`

**Issue:** The row asserts that `cardDependents` returns `DependentPackages`' result over a `lib` and `user` module.
`TestDependentPackages`' first row pins that result over the same shape, plus a tagged-test importer.
The `--card` row of `TestGateTest_Scopes` pins the same `./lib ./user` scope end to end.
The redundancy report lists the test as unique only because its failure row is unique.
The success row is a per-helper check the public-surface test already covers.

### [NIT] dependentsFor cleans a seed for lookup but not for de-duplication

**Location:** `internal/impactset/dependents.go:38`

**Issue:** `graph.byDir[path.Clean(arg)]` accepts `./lib/`, but `slices.Contains(result, arg)` compares against the raw `./lib/`, so `packageArgs` adds `./lib` a second time.
Today's only caller passes `planparser.CardPackages`' clean `./dir` spellings, so nothing breaks now.
The function's own handling is inconsistent, though, and the exported doc does not require clean arguments.

## Rejected

- Lens code-tests finding 3, that no test checks the fallback is visible to the caller: folded into F2, whose fix adds the assertion.
- Lens code-tests finding 1, the missing guard row for a dependent package: folded into F1, whose fix adds the row.
- The `cmd/lym`→`lymbin` rule not firing when `cmd/lym` joins as a dependent rather than a seed: matches the documented diff-scope rule, and none of the issue's packages has a test importing `lymbin`.
- The issue's `internal/shedrecipe` case: `internal/shedrecipe` imports `internal/loomengine`, so a card targeting `loomengine` now pulls `shedrecipe`'s tests in; the change description's claim holds for it.
- Scope: every goal point of the board entry has a change that builds it, and every change traces to one or is forced by one (lens code-scope reported none).

## Focus departures

- "Check also a card whose targets are non-Go files that a package embeds or reads, and whether the same graph rules as the diff scope apply to them." Not driven: I did not trace `planparser.CardTargetDirs` for a non-Go target in a directory without a `.go` file against the diff scope's nearest-enclosing-package rule. The `CardPackages` code shows that it seeds only directories that name or hold a Go file. The case stays open for the next round.
- "Whether every package `DependentPackages` adds can be handed to each listed tier's command and each vet run: packages whose files are all behind a build tag or platform constraint, test-only packages ..." Only partly driven. The listing runs under `tier0` through `tier3` and the host platform, so a package whose files are all behind a non-tier constraint is not listed and not added. I did not run a gate over a package whose files are all tier-tagged.
