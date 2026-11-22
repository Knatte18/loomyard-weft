---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/loomcli/hardenwiring.go:179
    summary: The new missing-board-entry refusal has no reaching test required by PATTERN-refusal-way-forward.
    fix: Extend TestWire_HardenerRecipe in internal/loomcli/wiring_harden_integration_test.go with present and absent board-task cases that invoke the wired campaign SeedBrief; assert the actual title and labels and the missing-entry message including its way forward.
    origin: lens:code-tests
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/shedadapters/bouncer.go:1928
    summary: A failed module lookup advances a title-only Hardener campaign with an empty charter and prevents the normal seed retry.
    fix: Make the Hardener seed path propagate brief or charter-production failure without synthesizing an empty focus or advancing to review; preserve ordinary Bouncer fallback behavior and add a Bouncer.Call case that retries successfully after the board lookup recovers.
    origin: lens:code-correctness
  - id: F3
    severity: MEDIUM
    class: design
    location: contracts/stencils/harden/harden-rubric-review.md:112
    summary: The judge must preserve the charter but its declared inputs never supply the current charter.
    fix: Supply the current charter path or its full contents to the Hardener judge through RenderJudgePrompt or the campaign facts, and require reading it before writing the next focus; keep other recipes unchanged and test the input delivery.
    origin: lens:code-affected-set
  - id: F4
    severity: MEDIUM
    class: scope
    location: internal/hardenshed/live.go:180
    summary: The copies tier remains an independent template default instead of being derived or folded as the board entry requires.
    fix: Derive the single copies tier from the resolved live selection or fold the copies count into harden-live; update the gate invocation and Preflight together, retain hub overrides, and test a hub that raises its live tier without separately setting the copies key.
    origin: lens:code-scope
  - id: F5
    severity: MEDIUM
    class: scope
    location: contracts/stencils/harden/harden-template-seed.md:93
    summary: The blanket multi-session test ban removes Crucible's exception for a campaign whose mission is the fan or cluster path itself.
    fix: Restore the mission-specific exception in the seed cost declaration while retaining named-test selection, declared session counts, and serial invocations; keep the rubric's execution-ban rule consistent with that exception.
    origin: lens:code-scope
  - id: F6
    severity: MEDIUM
    class: scope
    location: contracts/stencils/harden/harden-rubric-review.md:94
    summary: The required Crucible carry-over omits extending a maintained sandbox scenario suite when a newly found behavior is absent.
    fix: Carry the suite-maintenance rule into Hardener's fix instructions and rubric, including preservation of the coverage guard and the exception that no new suite or launcher is required when none exists.
    origin: lens:code-scope
  - id: F7
    severity: MEDIUM
    class: design
    location: internal/shedrecipe/entries_bouncer.go:368
    summary: No test checks that the recipe constructor passes the campaign brief into the seed prompt.
    fix: Add a fresh-seed case to TestHardenRoundKey that builds the Bouncer with harden_round and the named seed stencil, calls it through the fake shuttle, and asserts that the captured prompt contains the campaign's actual package or title-and-label brief.
    origin: lens:code-tests
  - id: F8
    severity: MEDIUM
    class: consistency
    location: internal/hardenshed/campaign_test.go:49
    summary: Removed package and hub-key refusal assertions have no Loosened-Check trailer explaining their removal.
    fix: Add a Loosened-Check trailer to a commit in the review range naming the removed refusal assertions and TestHubSetsGate, with observed before-and-after cases supporting their intentional retirement; preserve the requested new behavior instead of restoring obsolete refusals.
    origin: lens:code-tests
  - id: F9
    severity: MEDIUM
    class: design
    location: contracts/stencils/harden/harden-rubric-review.md:91
    summary: Hardener's new fixer obligations are placed in a rubric the fixer never receives.
    fix: Wire Hardener-specific fix instructions and the charter into the HardenFix prompt through entries_harden.go and the fix prompt composition path; deliver the live regression, sabotage, redeploy, teardown, package-doc and large-finding deferral rules there without changing other recipes' fixers.
    origin: handler
---

Reviewed the committed range `3a9f8d8419fbcd415992ca15a8c6e9d25822015e..489dfa7e56cb8a0a0b4bbbf6d006f7c72ad484ec`.
The decision record, its amendments, the board entry and the retired Crucible instructions supplied the scope.
Findings below are grounded in source and prompt tracing, not an end-to-end campaign reproduction.

### [BLOCKING] Missing refusal coverage

**Location:** `internal/loomcli/hardenwiring.go:179`

**Issue:** `TestWire_HardenerRecipe` checks that the campaign exists but never calls its new board lookup.
`TestCampaignSeedBrief` supplies a fake entry closure, so it cannot reach this refusal.
An absent board task is neither raw I/O propagation nor an unreachable wiring guard.
`PATTERN-refusal-way-forward` and `contracts/specs/refusal-spec.md` require a reaching message assertion.
The review instructions classify a listed PATTERN violation as BLOCKING.

### [MEDIUM] Failed charter generation becomes successful fallback

**Location:** `internal/shedadapters/bouncer.go:1928`

**Issue:** A title-only campaign depends on the new board lookup for its module identity.
If that lookup fails, `RenderSeedPrompt` returns an error, but `runSeedSpawn` logs it and returns nil.
`seedCall` then calls `ensureFocus(1)` and returns Stuck, which the recipe routes to review.
The resulting empty focus has no module identity or charter.
It also parses as an already-seeded focus, so restored board access does not trigger the seed again.
Both the correctness and affected-set lenses independently traced this same defect.

### [MEDIUM] Charter preservation has no input channel

**Location:** `contracts/stencils/harden/harden-rubric-review.md:112`

**Issue:** The new rule requires copying the standing charter sections into every subsequent focus.
The shared judge reads facts, the current review and the previous ledger.
Its supplied focus path is the next output, not the current charter.
Neither `bouncerfacts.go` nor `Campaign.RoundFacts` supplies the charter.
A valid review need not repeat the layouts, scope exclusions, session bans or teardown commands.
Thus a judge following its declared reading sequence lacks the material it must preserve.

### [MEDIUM] Copies still require an independent tier choice

**Location:** `internal/hardenshed/live.go:180`

**Issue:** The board entry explicitly requires deriving the copies tier or folding copies into the live key.
The diff keeps `harden-live-copies` independently resolved in `hardenPreflight` and invoked by `NewLiveGate`.
Changing only `harden-live` therefore leaves copies on the template's tier 0.
The landing summary acknowledges this departure; no amendment or Pattern-Departure trailer authorizes it.
The template default removes the initial write requirement but does not implement the requested derivation.

### [MEDIUM] Fan-focused campaigns cannot select their own test

**Location:** `contracts/stencils/harden/harden-template-seed.md:93`

**Issue:** The seed bans every test that spawns more than one real session.
The rubric then forbids executing any charter-banned test.
Crucible's required review template explicitly exempted a test when the round's mission was that fan or cluster path.
Without that exception, a campaign focused on the fan path cannot execute the relevant regression test.

### [MEDIUM] Existing scenario suites are not maintained

**Location:** `contracts/stencils/harden/harden-rubric-review.md:94`

**Issue:** Crucible's review template and README require extending a maintained sandbox suite for uncovered behavior.
They explicitly exempt creating a suite when none exists.
The new instructions require a live test and package-doc update but omit the existing-suite obligation.
The shared fixer instructions do not supply it either.
This misses the amendment's requirement to carry over the template and README rules.

### [MEDIUM] Recipe-level seed delivery is untested

**Location:** `internal/shedrecipe/entries_bouncer.go:368`

**Issue:** The adapter test directly sets `SeedBrief` and `SeedStencil`.
The recipe test checks the YAML value, while `TestHardenRoundKey` exercises later judge and halt seams.
None exercises the constructor-to-seed connection.
Dropping this assignment leaves those assertions intact but removes the required brief from the shipped seed.
The supplied mutation survivor is therefore a concrete missing assertion, not merely unreached code.

### [MEDIUM] Intentional check removals lack the required record

**Location:** `internal/hardenshed/campaign_test.go:49`

**Issue:** The diff replaces the no-package refusal assertion and deletes the hub-written-key refusal cases.
It also removes `TestHubSetsGate`.
The requested behavior supports those changes, but the rubric separately requires a Loosened-Check trailer.
None of the commit messages in the reviewed range carries one.
This finding requests the required evidence record, not reinstatement of retired behavior.

### [MEDIUM] Fixer instructions stop at the reviewer

**Location:** `contracts/stencils/harden/harden-rubric-review.md:91`

**Issue:** The new rubric tells the fixer to defer large fixes and leave live regression tests with package-doc updates.
The same rubric supplies sabotage, redeployment and teardown obligations.
However, `composeFixPrompt` in `internal/burlerengine/prompt.go` deliberately supplies no rubric or focus file.
The HardenFix recipe profile supplies source scope and language skills only.
The shared fixer prompt explicitly says it holds none of the reviewer's orientation.
A review that gives a source correction without repeating those standing rules leaves the fixer unaware of them.
The shared fix step also lacks Hardener's large-finding deferral rule.
Placing obligations in the review rubric does not deliver them to the separate fix session.

## Rejected

- Empty `seed_stencil` is not a construction defect: the optional accessor and field contract define it as the shared default.
- The unmapped sandbox wrappers are not uncovered solely because the reach report misses them; `TestSandboxVerbs` drives them.
- Marker and template-substitution assertions test rendering contracts, not prohibited assertions on static prose.
- The old refusal string in the Preflight test is an injected propagation fixture, not a stale production requirement.

## Validation

Attempted `timeout 3600s lym gate test --tier 0 ./internal/hardenshed ./internal/shedadapters ./internal/shedrecipe ./internal/loomcli`.
The command exited before tests ran because the filesystem denied writes to the hub's gate and runproc records.
No runtime pass is claimed, and no source file was changed.
The report format is checked separately with `lym burler validate-review`.
