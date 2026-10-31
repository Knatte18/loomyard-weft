---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/plan/08-one-loom-builder.md
    summary: Card 8's recipe-keyed New refuses a recipe without contract loom and stamps verify attempts only on verify_source tree_scope, but darn gains both keys only in card 11, so every darn build and card 8's own darn tests fail from card 8 to card 10
    fix: "In card 8, drop the non-loom refusal from New, and stamp DarnVerifyAttempts on the recipe whose build row engine is DarnWrite (the rows darnVerifyGatedRows names today) instead of on verify_source. Remove the sentence that calls the refusal a fail-loud guard with no reaching test. In card 11, which gives darn-recipe.yaml contract: loom and verify_source: tree_scope, add `internal/loomrecipe#New` under **Edit:** and `internal/loomrecipe/doc.go` under **Uses:**. Its Intent then has New refuse a recipe that does not declare contract: loom, naming NewPawl or lym shed, and switches the stamp onto verify_source: tree_scope. Card 11's TestNewDarn_Routing and the TestRecipeKeys rows already cover darn building under the new keys. An admissible alternative is to move card 11 ahead of card 8 with lym loom remove-card and insert-card."
    origin: lens:repo-invariants
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lym/plan/17-one-describe.md
    summary: "D8's bullet \"The description commit becomes one\" is carried by no card; darnDeps' Commit (internal/loomcli/darnwiring.go:152) still duplicates wire's CommitDescription (internal/loomcli/wiring.go:687)"
    fix: "In card 17, add `plan:internal/loomcli#commitDescription` -> `func commitDescription(location *lymcwd.Location) error` under **Create:** and `internal/loomcli#darnDeps` under **Edit:**. Add an Intent bullet: commitDescription commits the landing directory with the change-description subject; wire's CommitDescription and darnDeps' Commit both call it, so every loom recipe has one description commit (D8). Any existing CommitDescription and darn commit test keeps covering it."
    origin: handler
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lym/plan/13-one-discussion-validator.md
    summary: Card 13 drops DiscussionHistory from envkit nilLegal, but batten's wire never sets it and battencli's two-way intentionallyNil check lacks it, so battencli's tier0 TestWire_EverySeamFilled fails at card 13
    fix: "In card 13's Intent and test bullets, have `internal/battencli#intentionallyNil` (already a target) list `DiscussionHistory` with the reason that batten wires no loom record, beside `Darn.CheckRecord`."
    origin: lens:repo-invariants
  - id: F4
    severity: MEDIUM
    class: design
    location: _lym/plan/06-tapestry-config-module.md
    summary: Card 6 keeps LoadConfig from reading tapestry.yaml, but perchHardCap (internal/loomcli/circling.go:194-210), which card 6 targets, still reads cfg.Caps(PerchPlan) from that load, so the circling hard-cap check for Plan-Bouncer ignores a tuned tapestry.plan_hard_max_bounces
    fix: "In card 6's Intent, have `internal/loomcli#perchHardCap` (already a target) load LoadTapestryConfig and apply Config.WithTapestry before reading Caps when the row's perch is the plan perch. It routes the way forward through TapestryConfigSpec, as card 6 already says. Add a TestLoadTapestryConfig or circling-test row in which a tapestry.yaml plan_hard_max_bounces below loom's default is the cap perchHardCap returns for Plan-Bouncer."
    origin: lens:guard-reach
  - id: F5
    severity: MEDIUM
    class: scope
    location: _lym/plan/06-tapestry-config-module.md
    summary: "Card 6 misses readers of the moved keys: TestDispatchLayers sets loom's local plan_reviewer_model (internal/configcli/configcli_test.go:1306-1311), engineMinimalConfig names loom.implementer (internal/shedbuild/build_engines_test.go:62), and implementerEntry's way-forward texts cite loom.implementer (internal/shedrecipe/entries_implementer.go:30-41)"
    fix: "In card 6, add under **Edit:** `internal/configcli#TestDispatchLayers` (its row sets plan_reviewer_model on module tapestry), `internal/shedbuild#engineMinimalConfig` (implementer_setting tapestry.implementer, and the comment at build_engines_test.go:22 follows), `internal/shedrecipe#implementerEntry` (its example key becomes tapestry.implementer), and `internal/shedrecipe/doc.go`."
    origin: lens:affected-set
  - id: F6
    severity: MEDIUM
    class: scope
    location: _lym/plan/09-drop-tapestry-preflight.md
    summary: "Card 9 leaves docs naming the removed row or CheckSeed: VerifySeedOwnership's doc (internal/loomengine/seed.go:113-121), New's doc justifying the status-path check by tapestryPreflightEntry (internal/loomrecipe/loomrecipe.go:53-65), reportengine/doc.go:130 (only under Uses), the lockFree comment (internal/loomcli/smoke_test.go:214) and pattern/PATTERN-cwd-resolution.md:21"
    fix: "In card 9, move `internal/reportengine/doc.go` from **Uses:** to **Edit:**, and add under **Edit:** `internal/loomengine#VerifySeedOwnership`, `internal/loomrecipe#New`, `internal/loomcli#lockFree` and `pattern/PATTERN-cwd-resolution.md`. Their text drops Tapestry-Preflight and CheckSeed: the status-path check is kept for Shed's own status, and the cwd-resolution tiers end at preflight.Check. Card 8 lists `internal/reportengine/doc.go` under **Edit:** in place of **Uses:**, since an earlier Uses of a later target is refused (uses-later-target)."
    origin: lens:affected-set
  - id: F7
    severity: MEDIUM
    class: consistency
    location: _lym/plan/13-one-discussion-validator.md
    summary: pattern/PATTERN-gate-slot.md:28-29 names ValidateLoomDiscussion and ValidateDarnDiscussion as the discussion-gate pairs, which card 13 deletes, and no card lists the file
    fix: "In card 13, add `pattern/PATTERN-gate-slot.md` under **Edit:**. Its Pairs section names `loomshed.ValidateDiscussion` as the function both Discussion-Write's and Discussion-Burler-Fix's gates and `validate-discussion` call, and drops the Darn-Write discussion-gate pair, since Darn-Write's check is now a post-session guard."
    origin: lens:repo-invariants
  - id: F8
    severity: MEDIUM
    class: design
    location: _lym/plan/16-decision-add-by-rows.md
    summary: "The Not-committed-at-HEAD refusal names lym loom resume and says only a crash in Record-Write leaves it, but a darn run in flight that halted at Darn-Write on its old discussion gate also holds an uncommitted record; it resumes at Darn-Write, which commits no record after cards 13 and 15, so the refusal repeats"
    fix: "In card 16, make the Not-committed way forward depend on the current row. At Record-Write it names lym loom resume. On any later row it names `lym loom goto --to Record-Write`, then `lym loom resume`, which commits the existing pair without overwriting it, then re-running the verb. Add a TestDecisionVerb_Refusals row for an uncommitted record at current row Darn-Write. Since this widens D4's wording, ask the parent to record it with lym loom decision add."
    origin: lens:guard-reach
  - id: F9
    severity: MEDIUM
    class: decision
    location: _lym/plan/19-loom-verbs-loom-only.md
    summary: "Card 19 refuses a beam seed as naming no recipe this lym knows, with a re-seed way forward, where D2 and D11 (and D11's acceptance criterion) say lym loom refuses it as a non-loom recipe naming the lym shed verbs; the record carries no Added-after-Discussion entry for the departure"
    fix: "Keep card 19's beam text, since D11 also has lym shed refuse beam, so naming the lym shed verbs would point at a refusal (PATTERN-refusal-way-forward). Have the parent record the departure with lym loom decision add, naming the re-seed way forward for a seed whose recipe names no embedded recipe. Cite that entry in card 19's Intent."
    origin: handler
  - id: F10
    severity: LOW
    class: consistency
    location: _lym/plan/11-record-write-row.md
    summary: Card 11 adds Record-Write to darnPhases' setup but lists internal/reportengine/doc.go only under Uses, while reportengine/doc.go:132 still lists darn's setup as Preflight, Drift-Baseline and Drift-Await
    fix: "In card 11, move `internal/reportengine/doc.go` from **Uses:** to **Edit:**, and its darn setup sentence names Record-Write. This is admissible together with F6's change of card 8's Uses to Edit."
    origin: lens:repo-invariants
  - id: F11
    severity: LOW
    class: design
    location: _lym/plan/09-drop-tapestry-preflight.md
    summary: TestPreflight_BatcherCheck's cancelled-context row never reaches the cancel-wins branch after the batcher check, because preflightProducer.Call returns at entryErr first (internal/preflightshed/preflight.go:107-109)
    fix: "In card 9's test bullet, make the row's seam cancel the context and then return an error, so the call passes entryErr and reaches the cancel check after the batcher seam. Assert that the returned error is the cancellation error and that the outcome is not Stuck."
    origin: lens:guard-reach
  - id: F12
    severity: LOW
    class: scope
    location: _lym/plan/05-shed-keys-in-shed-only.md
    summary: internal/loomcli/doc.go:26,148,309 still say the driver_kind and arbiter keys come from loom.yaml or darn.yaml under a recipe override, which card 5 removes, and card 5 does not list the file
    fix: "In card 5, add `internal/loomcli/doc.go` under **Edit:**. The three sentences then say those keys come from shed.yaml and its layers, with a loom recipe's own values coming through its profile block's shed object."
    origin: lens:affected-set
  - id: F13
    severity: NIT
    class: consistency
    location: _lym/plan/11-record-write-row.md
    summary: Card 11 says the darn writer stops writing the record in card 14, but card 15 drops the stencil's Step 3; card 14 is the shared writer wrapper
    fix: "In card 11's Intent, change \"until card 14 stops it writing one\" to \"until card 15 stops it writing one\"."
    origin: handler
---

### [BLOCKING] Card 8's builder refuses darn until card 11

**Location:** `_lym/plan/08-one-loom-builder.md`
**Issue:** Card 8's `New` refuses a recipe that does not declare `contract: loom`, and it stamps `DarnVerifyAttempts` only for `verify_source: tree_scope`. `contracts/recipes/darn-recipe.yaml:61-66` carries neither key. Card 11 adds both (card 3 says so too: "Darn declares the contract in card 11"). From card 8 through card 10, `New(shedrun.RecipeDarn, …)` refuses every darn build, so the darn tests card 8 rewrites (`internal/loomrecipe/darnrecipe_test.go:38,131,251,309`) and `TestWire_DarnRecipe` fail at card 8's own gate. This breaks the plan spec's "Plan vs. schedule" rule, since card 8 depends on card 11. The card also calls the refusal unreachable, a fail-loud guard with no reaching test, but darn reaches it, and card 19's arm-path refusal does not exist yet at card 8. The guard-reach lens raised the same defect.

### [MEDIUM] D8's single description commit is uncarried

**Location:** `_lym/plan/17-one-describe.md`
**Issue:** D8 lists "The description commit becomes one." No card carries it: the plan never mentions `CommitDescription` or the darn landing commit as something to merge. `darnDeps.Commit` (`internal/loomcli/darnwiring.go:152-155`) is a copy of wire's `CommitDescription` (`internal/loomcli/wiring.go:687-690`); its own comment says "Commit mirrors CommitDescription". The decision record has no `Added after Discussion` entry dropping this step.

### [MEDIUM] Batten's seam test breaks when DiscussionHistory leaves nilLegal

**Location:** `_lym/plan/13-one-discussion-validator.md`
**Issue:** Card 13 removes `DiscussionHistory` from `internal/testkit/envkit/envkit.go:54`. Batten's wiring never sets it, and battencli's `TestWire_EverySeamFilled` (`internal/battencli/wire_seams_test.go:184-193`, tier0) checks `intentionallyNil` both ways. Card 13 adds only `Darn.CheckRecord` there. The affected-set lens raised the same defect.

### [MEDIUM] The plan perch's circling hard cap ignores tapestry.yaml

**Location:** `_lym/plan/06-tapestry-config-module.md`
**Issue:** `perchHardCap` (`internal/loomcli/circling.go:194-210`) loads `loomengine.LoadConfig(dirs, "loom")` and returns `cfg.Caps(perch).Hard`. Card 6 makes `LoadConfig` never read `tapestry.yaml` and applies `WithTapestry` only in `wire` and `reviewBudget`. It changes only `perchHardCap`'s way-forward route, so for Plan-Bouncer the cap stays the shared fallback whatever `tapestry.plan_hard_max_bounces` holds.

### [MEDIUM] Card 6 misses three readers of the moved keys

**Location:** `_lym/plan/06-tapestry-config-module.md`
**Issue:**
- `internal/configcli/configcli_test.go:1306-1311` (`TestDispatchLayers`) writes `plan_reviewer_model` to loom's local layer. That key leaves loom's template.
- `internal/shedbuild/build_engines_test.go:62` (`engineMinimalConfig`) sets `implementer_setting: loom.implementer`, which envkit's template lookup will no longer answer.
- `internal/shedrecipe/entries_implementer.go:30,33,41` gives `loom.implementer` as the example in three way-forward texts.

Each was verified by reading the lines.

### [MEDIUM] Card 9 leaves stale docs naming Tapestry-Preflight and CheckSeed

**Location:** `_lym/plan/09-drop-tapestry-preflight.md`
**Issue:** These still name the removed row or `CheckSeed` after card 9, and card 9 lists none of them as an edit target:
- `internal/loomengine/seed.go:113-121`, `VerifySeedOwnership`'s doc.
- `internal/loomrecipe/loomrecipe.go:53-65`, `New`'s doc on `tapestryPreflightEntry`.
- `internal/reportengine/doc.go:130`, which card 9 holds under Uses only.
- `internal/loomcli/smoke_test.go:214`, the `lockFree` comment.
- `pattern/PATTERN-cwd-resolution.md:21`, the third tier `loomengine.CheckSeed`.

The repo-invariants lens raised the reportengine and cwd-resolution items independently. Card 8's `**Uses:**` on `internal/reportengine/doc.go` blocks a later card from editing it, so the fix moves that entry to card 8's Edit list. The overlay run below confirms the move.

### [MEDIUM] PATTERN-gate-slot background still names the two removed validators

**Location:** `_lym/plan/13-one-discussion-validator.md`
**Issue:** `pattern/PATTERN-gate-slot.md:28-29` pairs Discussion-Write's gate and `validate-discussion` with `loomshed.ValidateLoomDiscussion`, and the Darn-Write `discussion` gate with `loomshed.ValidateDarnDiscussion`. Card 13 deletes both and the darn gate itself, and no card updates the background file of the PATTERN entry D5's constraint names.

### [MEDIUM] Decision add's not-committed way forward fails for a run in flight at Darn-Write

**Location:** `_lym/plan/16-decision-add-by-rows.md`
**Issue:** Today `NewDarnDiscussionGate` commits the record only on a clean check (`internal/loomshed/gates.go:213-229`), and Darn-Write's own commit covers the landing directory alone (`internal/loomcli/darnwiring.go:152-155`). A darn run that halted on gate exhaustion before landing therefore holds a written but uncommitted record and is parked on Darn-Write. Card 16 names `lym loom resume` for this state and claims only a crash inside Record-Write leaves it. Resuming at Darn-Write never commits the record once cards 13 and 15 land, so the refusal repeats. D4 states the same premise, so the fix goes through the parent.

### [MEDIUM] Card 19's beam refusal departs from D2 and D11 without a record

**Location:** `_lym/plan/19-loom-verbs-loom-only.md`
**Issue:** D11 says "`lym loom` refuses it as a non-loom recipe (D2)", D2 names the `lym shed` verbs as the way forward, and the D11 acceptance criterion repeats it. Card 19 gives a seed recording `beam` its own refusal, "names no recipe this lym knows", with a re-seed way forward. The card's text is the sounder one, since `lym shed` refuses `beam` as unknown. But it overturns a decided way forward with no `Added after Discussion` entry, which the parent must record.

### [LOW] Card 11 leaves reportengine's darn setup line stale

**Location:** `_lym/plan/11-record-write-row.md`
**Issue:** `internal/reportengine/doc.go:132` lists darn's setup phase as Preflight, Drift-Baseline and Drift-Await. Card 11 adds Record-Write to `darnPhases` setup but keeps that doc under Uses.

### [LOW] TestPreflight_BatcherCheck's cancel row cannot reach its branch

**Location:** `_lym/plan/09-drop-tapestry-preflight.md`
**Issue:** `preflightProducer.Call` checks `entryErr(ctx, …)` first (`internal/preflightshed/preflight.go:107-109`). A context cancelled before the call returns there, before the reconcile and the batcher seam, so the row pins the entry check rather than the cancel-wins branch after the batcher check.

### [LOW] Card 5 leaves loomcli's doc naming the recipe override

**Location:** `_lym/plan/05-shed-keys-in-shed-only.md`
**Issue:** `internal/loomcli/doc.go:26` says an unseeded run's driver is "`loom.yaml`'s `driver_kind`" resolved over `shed.yaml`'s. Line 148 says the arbiter keys come "under the run's recipe override, `darn.yaml`'s for a darn run and `loom.yaml`'s otherwise". Line 309 says "the recipe's driver_kind override". Card 5 removes that override and does not list the file.

### [NIT] Card 11 cites the wrong card

**Location:** `_lym/plan/11-record-write-row.md`
**Issue:** "The darn writer still finds a record that validates and leaves it, until card 14 stops it writing one." Card 15 drops `darn-template-write.md`'s Step 3. Card 14 is `shared-writer-wrapper`.

## Proposed-fix check

`lym loom validate-plan --overlay` ran with `{}` as the baseline and reported only the informational `create-new-unit` on card 6. A combined overlay then carried the target changes of F1, F2, F5, F6, F7, F10 and F12: cards 5, 6, 8, 9, 11, 13 and 17. In that combined run, card 6 lists `internal/shedrecipe/doc.go` under Edit and card 8 lists `internal/reportengine/doc.go` under Edit. It reported no blocking finding beyond the baseline. An earlier combined run with `internal/shedrecipe/doc.go` under card 6's Uses, and card 8's reportengine Uses left in place, hit `uses-later-target`. The fixes above are written in the admitted form.

## Focus directives settled

- Card cross-references: card 15's "card 13's `CheckRecord`" names the `DarnDeps.CheckRecord` field card 13 adds, so it is correct. Card 11's "card 14" is wrong (F13). Card 3's "darn declares the contract in card 11" is right, and is the root of F1.
- Config and recipe chain: card 5 runs after card 4's `RetiredWayForward`, and card 9 edits `RetiredRows`/`TestRecipeKeys` after card 7 creates them, so both hold. Card 7's `IsLoom` reads darn as not loom until card 11, and its test does not claim otherwise. Card 8 is the order break (F1).
- Fidelity walk:
  - D1: cards 3 and 11. D2: cards 7 and 19. D3: cards 16, 19 and 20, plus card 15's stencil for `## PATTERN changes`. D4: cards 1, 11, 15 and 16. D5: card 13. D6: card 15. D7: card 12. D8: cards 8, 14, 17 and 18, but the description commit is uncarried (F2). D9: card 10. D10: card 9. D11: cards 2 and 19. D12: card 6. D13: cards 4 and 5. D14: no card owed. The Added-after-Discussion entry: card 3.
  - D4's three darn `decision add` refusals are carried by card 16, with F8 on the not-committed case. D5's guard-failure recovery text is in card 13. D9's in-flight goto note is in card 10's recipe header. D12's profile and `run_settings` translation is in card 6 through `KeyMoves` and `moveRunSettings`.
  - Constraints: gate-slot parity is card 13, with the background doc stale (F7). Sparse-hub-base is card 6. Tolerant records is card 2.
- Fidelity, other direction: every extra target in cards 5, 6 and 9 is a reader, fixture or doc of the decided change, licensed by the Scope paragraph's stale-docs clause or by the decision it serves. None adds unlicensed scope.
- Reachability:
  - Card 3's clauses are reachable past `Parse`.
  - Card 5's `PrepareRunSettings` refusal runs after `moveRunSettings`, which moves none of the seven keys, and before any `Migrate` that could strip them.
  - Card 15's halts are reachable.
  - Card 16's rows are reachable, with F8.
  - Card 19's non-loom refusal is reachable for a `beam` seed, since `ReadSeed` validates no recipe (`internal/shedrun/seed.go:104-146`).
- Attack surface: every card the directive named states its bound. Card 7 says `ServedByLoom` admits pawl alone. Card 9 has its Bound paragraph. Card 15 says the halt lets nothing through. Card 19 has its Bound paragraph and the `checklist` exemption. Card 5's change only adds refusals.
- Research steps: no card leaves a design fact to be found while it runs or carries a halt-on-discovery rule.
- Granularity and test economy:
  - Card 10's `TestRecipe_DriftVerifyPrecedesCodeReview` pins tapestry `on_done` edges, which no existing test pins: `TestNew_ShapeMatchesRecipe` checks reachability only.
  - Card 18's `TestRecipeKeyedWiring` covers branches no existing loomcli test drives.
  - Card 6's mirrored loader symbols are the module's public surface, which `configreg.Modules` registers.
- Member glyphs and re-sign arrows: the arrows are present on cards 13 and 16. Card 6's `LoadConfig` keeps its signature. Card 8's package glyph is rejected below.
- ImpactSummary: each Edit and Delete card's line states a blast-radius conclusion.
- Verify lines and board entry: no card carries a `**Verify:**` line. No card writes or promotes a board entry. Card 11 reads the board, and card 5 changes how `run_settings` is checked, not the entry.

## Rejected

- Card 8 names the package glyph `internal/loomrecipe#` while its Intent re-signs `New` and `Routing`. The plan stencil admits a package glyph for a card that changes the whole package, the re-sign-arrow rubric item fires only on a member glyph, and listing members beside the package is refused by `redundant-package-target`. Not a finding.
- Affected-set lens, card 9: `internal/shedrecipe/conformance_test.go:36` and the `internal/loomcli/bootstrap.go:142` comment are already covered by card 9's `internal/shedrecipe#minimalEntries` and `internal/loomcli#awaitRunLock` targets, so they are dropped from F6.
- Affected-set lens, card 5: `internal/sheddriver/config_test.go:104` passes "loom" only as a module label to `CheckOverride`, which batten still uses, and needs no change. It is dropped from F12.
- Guard-reach lens finding 1 (card 8's refusal is reachable) is the same defect as F1, merged there.
- Affected-set lens finding 1 duplicates F3, and repo-invariants findings 4 and 5 are folded into F6 and F10.
