---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/loomshed/recordseed.go:47
    summary: "PATTERN-code-conventions (Named constants): the seed param name parent gets a second definition, the private loomshed constant seedParentParam, and a new bare literal at internal/loomcli/wiring.go:569 (seed.Params[\"parent\"]), beside the existing literal at internal/loomcli/sharedbootstrap.go:293."
    fix: "In internal/loomshed/recordseed.go rename the constant seedParentParam to the exported SeedParentParam, with a doc comment saying it is the seed parameter that records a loom run's parent branch, and use it in recordSeedStuckReason. In internal/loomcli/wiring.go:569 read seed.Params[loomshed.SeedParentParam]. In internal/loomcli/sharedbootstrap.go loomSeedFor, write Params: map[string]string{loomshed.SeedParentParam: parent}. Both loomcli files already import internal/loomshed (wiring.go names loomshed.RecordSeedDeps, sharedbootstrap.go calls loomshed.Seed), and loomshed imports no shedrun, so no new import edge is needed."
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:code-tests
    location: internal/loomshed/recordseed_test.go:331
    summary: "PATTERN-refusal-way-forward: the new refusal raised when the seed read fails (loomshed: Record-Seed: read the run's seed: ...; way forward: the fault is transient, re-step the Record-Seed row) is reached by ReadSeedErrorIsReturned, but the test asserts only errors.Is and never the message."
    fix: "In TestRecordSeed_Call's ReadSeedErrorIsReturned subtest, keep the errors.Is check and also assert that err.Error() contains both \"read the run's seed\" and \"way forward: the fault is transient, re-step the Record-Seed row\", the text recordWrite.fail renders at internal/loomshed/recordwrite.go:237 for p.name Record-Seed."
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: contracts/specs/shed-recipe-spec.md:224
    summary: "The shed recipe spec still states the old rules the diff changes. Line 208 says an outermost block row must set on_done, but D3 admits a last always-gated one. Clause 1 (line 224) requires exactly one record row, and the record-only paragraph says clause 1 is its whole contract, with no seed row, LoomSeedEngine or seed-after-record order. Line 12 names tapestry-recipe.yaml as the file holding each row."
    fix: "In contracts/specs/shed-recipe-spec.md: at line 208, replace 'and an outermost block row must set on_done' with the rule expand.go's doc states: an outermost block row with an empty on_done is admitted only as its recipe's last row with block_gate always, and its inlined rows then keep an empty on_done. Rewrite clause 1 at line 224 as: at most one record row (an engine of shedrecipe.LoomRecordEngines), at most one seed row (shedrecipe.LoomSeedEngine) and at least one of the two, with the seed row after the record row on the on_done path from entry when both are present; a seed row stands for the record row in every later clause when there is no record row. Leave the record-only paragraph's 'clause 1 is its whole contract' as it is, since clause 1 now covers the seed row too. At line 12, say the recipes are the data files under contracts/recipes, tapestry-recipe.yaml composing motif-recipe.yaml and jacquard-recipe.yaml through block rows."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/loomcli/wiring.go:552
    summary: "No test pins recordSeedDeps, which fills the values Record-Seed's Stuck reason renders. Every test stubs the seams, so a wrong param key, recipe or seed path in ReadSeed, or a wrong slug, would pass. Those are Recipe tapestry, seed.Driver, seed.Params parent, shedrun.SeedFile(l, runID) and seedSlug."
    fix: "Add a tier-0 test in internal/loomcli, beside the other wiring tests, that builds a *lymcwd.Location for a temp worktree and writes a seed with shedrun.WriteSeed(l, shedrun.SelfRunID, shedrun.Seed{Recipe: shedrun.RecipeJacquard, Driver: shedrun.DriverGo, Params: map[string]string{\"parent\": \"feature-x\"}}). It then calls recordSeedDeps(l, record, shedrun.SelfRunID, \"\") and asserts ReadSeed returns ReseedValues{Recipe: \"tapestry\", Driver: \"go\", Parent: \"feature-x\"}, SeedPath equals shedrun.SeedFile(l, shedrun.SelfRunID), and Slug equals the worktree's slug literal. Unverified: which location fixture the loomcli wiring tests use (battencli's tests use locationkit.Location(hub, slug, \".\")). Reuse that kit if it builds a loomcli location; otherwise build the Location the way the existing wiring_*_test.go files do."
  - id: F5
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/landing/summary.md:67
    summary: "The Merge danger call of two-way with blast radius loom understates what a revert breaks. Seeds may now record recipe jacquard, which a binary without this change refuses in shedrun.ValidateRecipe. In-flight tapestry runs may record current_producer Jacquard or Record-Seed, rows that a reverted tapestry recipe lacks. batten's overview stage numbers shift too, and the shed recipe table gains a recipe, so the radius is wider than loom."
    fix: "In _lym/landing/summary.md's Merge danger section set the door to one-way, naming the two records another binary reads: a seed recording recipe jacquard, and a tapestry run's status naming the Jacquard or Record-Seed row. Set the blast radius to loom, shed and batten."
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/recipes/arras-recipe.yaml:15
    summary: "The arras header says its rows follow the shared-segment ring that tapestry-recipe.yaml's header states, but the diff moved that ring rule out of the tapestry header into jacquard-recipe.yaml's header."
    fix: "In contracts/recipes/arras-recipe.yaml line 15, replace 'tapestry-recipe.yaml's header states' with 'jacquard-recipe.yaml's header states'."
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/loomshed/loomshed.go:5
    summary: "The loomshed comments still say tapestry-recipe.yaml holds the rows. Those are loomshed.go:5, loomshed.go:12 and doc.go:3 (Loom's ordered producer list lives in contracts/recipes/tapestry-recipe.yaml). Tapestry now holds only the Motif and Jacquard block rows, and the rows live in motif-recipe.yaml and jacquard-recipe.yaml."
    fix: "In internal/loomshed/loomshed.go line 5, say the names are verbatim per the rows of the loom recipes under contracts/recipes. At line 12, say those recipe files spell the same names as YAML strings. In internal/loomshed/doc.go line 3, say loom's producer lists live in the recipe files under contracts/recipes, tapestry's composed from the motif and jacquard recipes through block rows."
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/recipes/jacquard-recipe.yaml:263
    summary: "The comment moved with Plan-Burler-Review, 'fix-scope: overlay, matching the Discussion-Burler-Review row above', names a row the jacquard file does not hold."
    fix: "In contracts/recipes/jacquard-recipe.yaml line 263, replace 'matching the Discussion-Burler-Review row above' with 'matching motif-recipe.yaml's Discussion-Burler-Review row'."
  - id: F9
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/loomshed/recordwrite.go:211
    summary: "In Record-Seed's stored-only mode, a run whose record exists but whose support log is missing and whose entry has no stored record gets boardSupportLog. That log says the record was written from the board entry with no interview, which is false for a row that never reads the body."
    fix: "In recordWrite.Call's logMissing branch, when p.noStoredReason is non-nil and storedFound is false, write a support log that makes no claim about the record's source. Add a loomshed function beside StoredRecordSupportLog that renders the same four headings with the interview line 'The support log was missing when the record was seeded; the record was kept as it stood.'. Add a recordseed_test.go row in which the record exists, the support log is missing and ReadStored finds nothing, asserting that log's bytes and an unchanged record."
  - id: F10
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shedbuild/expand_test.go:108
    summary: "No test pins the nested run-ending case the summary claims. A block row inside the last always-gated block whose own exit stays empty must itself be its sub-recipe's last row with block_gate always."
    fix: "Add two rows to TestExpand in internal/shedbuild/expand_test.go. Both use a parent whose last row is an always-gated block with no on_done over a sub-recipe that itself holds a block row with no on_done. In the admitted row the inner block row is the sub-recipe's last row with block_gate always; assert the expanded rows, the innermost last row keeping an empty on_done. In the refused row the inner block row is not last; assert the error 'shedbuild: block \"<inner>\": on_done is empty; way forward: set on_done to the row the run takes after the block'."
  - id: F11
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomshed/recordseed_test.go:302
    summary: "The cancel check the diff adds before the stored-only Stuck (recordwrite.go:185) has no test. Removing it would pass every test."
    fix: "Add a subtest to TestRecordSeed_Call with no stored record and DiscussionRow Discussion-Write, calling Call with an already-cancelled context. Assert that the returned error is the cancellation error, the way recordwrite_test.go's CancelledContextWins asserts it for Record-Write, that the outcome is not Stuck, and that neither file exists."
  - id: F12
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/landing/summary.md:45
    summary: "The change description names the tooling's task record as the authority for its scope ('The record limits this change to Record-Seed's engine and contract steps'), which the Names of the tooling rule forbids."
    fix: "In _lym/landing/summary.md line 45, replace 'The record limits this change to Record-Seed's engine and contract steps, and leaves those refusals, batten's lists and the report's phase tables to later changes.' with 'This change covers Record-Seed's engine and contract steps only; those refusals, batten's lists and the report's phase tables are later changes.'"
---

The diff makes the change the amended board entry asks for.
The jacquard rows from Drift-Baseline on and its top-level keys are byte-identical to the base tapestry file, checked by `diff`.
Tapestry is the two block rows.
Expand's run-ending rule, the widened loom contract, the Record-Seed producer and its registry entry, the wiring and `RecordEngine` behave as D1–D4's engine and contract steps ask.
The Stuck reason's verbs and flags exist (`lym batten cancel --remove`, `lym shed seed self --recipe --driver --param`, `lym loom goto --to`), and the re-seed sequence moves the run back from Record-Seed to Discussion-Write under the re-seeded tapestry.
Two constraint breaches block the round: a repeated seed-param literal, and a new refusal whose message no test asserts.

### [BLOCKING] The seed's parent param name is defined twice and repeated as a literal

**Location:** `internal/loomshed/recordseed.go:47`, `internal/loomcli/wiring.go:569`, `internal/loomcli/sharedbootstrap.go:293`

**Issue:** `PATTERN-code-conventions` requires a name that can go stale to be one named constant, declared once and referenced everywhere.
The diff declares a private `seedParentParam = "parent"` in loomshed and adds the bare literal `seed.Params["parent"]` in the wiring, while the writer at `loomSeedFor` keeps its own literal.
If the key were renamed at one site, the reason would render a wrong parent without failing.

### [BLOCKING] Record-Seed's seed-read refusal has no test asserting its message

**Location:** `internal/loomshed/recordseed_test.go:331`

**Issue:** `PATTERN-refusal-way-forward` requires a new refusal in a loom package to land with a test that reaches it and asserts its message.
`ReadSeedErrorIsReturned` reaches the new "read the run's seed" refusal but checks only `errors.Is`, so its way-forward text is unpinned.

### [MEDIUM] The shed recipe spec still states the old block and loom-contract rules

**Location:** `contracts/specs/shed-recipe-spec.md:224`

**Issue:** The spec is the repository document for the recipe format and the loom contract.
After this diff it contradicts `Expand`, which now admits a last always-gated block row with no `on_done`.
It also contradicts `checkLoomContract`, which now admits a seed row beside or in place of the record row.
It still names `tapestry-recipe.yaml` as the file that holds each row.

### [MEDIUM] No test pins the Record-Seed wiring

**Location:** `internal/loomcli/wiring.go:552`

**Issue:** `recordSeedDeps` maps the real seed onto the values the jacquard Stuck reason prints, and it sets the slug and the seed path.
Every test stubs these seams, so a wrong key or path in the wiring would ship a way forward that does not work.

### [MEDIUM] The Merge danger call understates what a revert breaks

**Location:** `_lym/landing/summary.md:67`

**Issue:** The diff adds a recipe name that seeds may record and that an older binary refuses.
It also changes tapestry's graph, so an in-flight run may record the `Jacquard` or `Record-Seed` row, which a reverted recipe lacks.
These are records another binary reads, so the `two-way` call does not hold.
The `loom` radius also leaves out shed's recipe table and batten's stage display.

### [LOW] The arras header points at a tapestry header that no longer states the ring rule

**Location:** `contracts/recipes/arras-recipe.yaml:15`

**Issue:** The shared-segment ring rule moved from the tapestry header to the jacquard header, so arras's pointer now leads to a header that no longer states it.

### [LOW] The loomshed comments say tapestry-recipe.yaml holds the rows

**Location:** `internal/loomshed/loomshed.go:5`

**Issue:** `loomshed.go:5`, `loomshed.go:12` and `doc.go:3` describe tapestry's file as the row list, but tapestry now holds only the two block rows.

### [LOW] A moved comment names a row the jacquard file lacks

**Location:** `contracts/recipes/jacquard-recipe.yaml:263`

**Issue:** "matching the Discussion-Burler-Review row above" was true in tapestry's file, but it is false in jacquard's, which holds no Discussion row.

### [LOW] The stored-only mode can write a support log that misstates the record's source

**Location:** `internal/loomshed/recordwrite.go:211`

**Issue:** This case arises when the record exists, the support log is missing and there is no stored record.
Record-Seed then writes the board-rendered log, which claims the record came from the board entry.
A seed row never renders from the body, so the claim is false.

### [LOW] The nested run-ending block case is untested

**Location:** `internal/shedbuild/expand_test.go:108`

**Issue:** The summary's design decision states that a block nested in a run-ending block is held to the same last-and-always rule.
`expandRows` implements this, but no `TestExpand` row pins it.

### [LOW] The cancel check before Record-Seed's Stuck is untested

**Location:** `internal/loomshed/recordseed_test.go:302`

**Issue:** The stored-only Stuck branch checks for cancellation before it returns, but no Record-Seed test covers a cancelled context.

### [LOW] The change description names the task record as its scope authority

**Location:** `_lym/landing/summary.md:45`

**Issue:** "The record limits this change ..." names the tooling's decision record, which the Names of the tooling rule bars from the change description.

## Rejected

- The jacquard recipe missing from batten's child-recipe list (`battencli/wire.go:611`) and the report tables (`reportengine/runs.go`, `timingspans.go`) is no finding. The amended entry takes no batten or report work, and the summary names both as later changes.
- `decision.go`'s `rowWritten` reads false for jacquard, and the RecordWrite-path refusals still name Record-Write. This is no finding: the amended entry takes only D4's engine and contract steps, not the decision-verb step.
- The `Loosened-Check:` trailer on `2ff32d728` is not a loosening to judge. It covers new intentionally-nil seams in batten's seam test, not a widened assertion on existing behaviour.
- The changed Preflight row in `discussionapproval_test.go` (false to true) and the shifted stage numbers in the battencli and loomrecipe tests follow the new graph D2 asks for. They are not loosened checks.
