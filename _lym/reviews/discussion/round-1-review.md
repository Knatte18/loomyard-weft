---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md, D4 first paragraph
    summary: Writing the board body verbatim under Goal turns the body's own H2 headings into record sections, so decision add, carry-over and the PATTERN parse can act on a body heading such as Decisions, Open risks or PATTERN changes
    fix: In D4, state how Record-Write embeds the body so no line of it starts an H2 the parser reads, by demoting every body heading one level or by fencing the body (fenced lines are not read per the discussion spec), and state the chosen form in D4's Bound line; add to D4's acceptance criterion that a body holding a `## Decisions` and a `## Open risks` heading yields a record whose `decision add` append lands under the record's own `## Decisions`.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md, D10
    summary: D10 drops the half-finished verdicts of CheckSeed (history past the Preflight rows, a set start_sha, pause_requested) without stating what a half-finished seed reaching the row after Preflight can now do or what bounds it
    fix: In D10, name the dropped half-finished checks explicitly beside the stat race guard, and add a Bound line stating what they let through (a run with history past Preflight, a set start_sha or pause_requested proceeding into Discussion-Write) and what bounds it, citing the code that makes the bound hold, such as the only route back to Preflight being a goto re-entry the old check already waived and the seed verbs refusing to overwrite an existing seed; if no such bound holds in code, keep the half-finished check in Preflight for a recipe with a record row.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md, D13
    summary: D13 promises a refusal for retired arbiter, tackler and driver-kind keys, but the repo's retired-key route admits a CategoryNone key and drops it on every load, and the local loom.yaml or darn.yaml Seed-Child already wrote in seeded pairs would silently lose its overrides
    fix: In D13, name the mechanism that refuses the removed keys, as a refusal in `PrepareRunSettings` for these keys under `loom` or `darn` rather than a `RetiredKeys` entry, or `RetiredKeys` with a category that refuses; state the real refusal points (board run_settings writes, every `profiles.yaml` write and Seed-Child) in place of when the run starts; and state what an already-seeded pair whose local loom.yaml or darn.yaml holds one of the keys does after landing (refused at load with the way forward, or translated to the local shed.yaml), adding a matching line to D13's acceptance criterion and to the Open risks line on profiles.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md, D4 second paragraph
    summary: The in-flight darn halt is not reached as stated and its way forward fails, since the discussion gate runs only after a full writer session, its Stuck reason carries the resume way forward rather than goto Record-Write, a goto to Record-Write leads back into Drift-Baseline which refuses once HEAD is past the branch point, and runs parked on Drift-Baseline or Drift-Await skip Record-Write entirely
    fix: Rewrite D4's in-flight paragraph to state the handling per parked row, Drift-Baseline, Drift-Await and Darn-Write, choosing one mechanism that holds in code, for example the retired-row data or a pre-session check in Darn-Write that refuses a missing record before spawning the writer with a reason naming the way forward, and a way forward whose route does not pass Drift-Baseline after code is committed (such as `lym loom goto --to Record-Write` with Record-Write's `on_done` skipping drift for a run past its baseline, or the orch running Record-Write's write directly); update the D4 acceptance criterion and the Open risks line to the chosen handling.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md, D6
    summary: D6 does not place the no-commit check relative to the Darn-Write handoff gates, so a stop report the description gate refuses exhausts that gate and halts with its findings rather than the change description, and a respawn holding earlier commits past the fork never reaches the no-commit halt
    fix: In D6, state that the stop is detected before the handoff gates run or that the description gate accepts a stop report (and in which form, per the final-summary spec), and replace the no-commit-past-the-fork test with a signal that holds on a respawn with prior commits, such as a stop marker the writer writes or no commit made by this spawn; update D6's Bound line and acceptance criterion to match.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md, D6 and D7
    summary: With no parent, a Decision-for note's answer never enters the record, because D6 forbids the writer to edit it and decision add refuses a task session, while D7 makes the record the Code-Review answer key, so review judges a note-following change against a record without the note
    fix: In D6, state how a no-parent answer reaches the record, for example the halt's way forward naming `lym loom decision add` run by the orch or operator before `lym loom resume`, so the resumed writer and the perch both read it as an Added after Discussion entry; add that to D6's acceptance criterion.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md, D4 Bound line
    summary: Record-Write skips when a record exists, so a crash after writing the record but before the support log or the commit leaves a pair that is never completed or committed, since D5 makes Darn-Write's gate validate-only
    fix: In D4, change the skip test to a valid committed pair, so Record-Write writes whichever file is missing and commits when the pair is uncommitted, and still never overwrites a present record; state this in the Bound line and add a crash-between-writes row to D4's acceptance criterion.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md, D12
    summary: D12 keeps the plan keys' fallback to loom.yaml but not the check that a perch's resolved hard cap is not below its resolved budget, which LoadConfig makes from one loom Config today and which a per-module tapestry.yaml load cannot see
    fix: In D12, state where the plan perch's budget and hard cap are resolved and validated once the plan keys sit in tapestry.yaml (a load that reads both modules, named), so the hard-cap-not-below-budget refusal still covers the plan perch, and add a row to D12's acceptance criterion with tapestry.plan_hard_max_bounces below the inherited loom.review_max_bounces being refused.
  - id: F9
    severity: MEDIUM
    class: scope
    origin: handler
    location: _lym/discussion/decision-record.md, Scope In
    summary: Scope In names package docs, overview, recipe headers, stencils and the rubric but no spec, while D5, D1, D8 and D10 contradict pinned specs that would stay stale after landing
    fix: Add to Scope In that the pinned specs the decisions change are updated with them, naming each, so the plan writer carries them, namely `contracts/specs/loom-discussion-spec.md` (the history check runs on every loom record, no longer only darn, and citation and relevant checks follow the record row's engine), `contracts/specs/shed-recipe-spec.md` (the top-level `contract` and `verify_source` keys) and `contracts/specs/loom-status-spec.md` (the Tapestry-Preflight row and `CheckSeed`'s verdicts).
  - id: F10
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md, PATTERN changes, PATTERN-shed-stop-chain
    summary: The declared PATTERN-shed-stop-chain text drops the current with module overrides and says arbiter keys live in shed.yaml, while D13 keeps batten.yaml's arbiter and tackler overrides, so the landed rule would read batten.yaml as a violation
    fix: Change the declared text's last clause to admit batten's module override, for example arbiter keys in shed.yaml, batten's overridable in batten.yaml, a loom recipe's own through its profile block, keeping the rule text within the PATTERN rule-text cap, and make the Constraints line and Notes match.
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md, D13 against Scope Out and D2
    summary: D13 moves pawl's override source from loom.yaml to shed.yaml, pawl included, while Scope Out says every pawl path stays and D2 says pawl keeps every wiring path it uses today
    fix: Add to Scope Out's pawl bullet and to D2's exception sentence that D13's override move applies to pawl too, by parent decision, as the one pawl path this task changes.
  - id: F12
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md, D13
    summary: Two existing paths still write the keys D13 retires, KeyCopies copying loom.arbiter keys into darn and loom's migrateShedOverrides rewriting driver_kind and the tackler keys, and D13 is silent on both
    fix: Add to D13 that the three loom-to-darn `arbiter*` entries in `KeyCopies` in `internal/configreg/keymove.go` and loom's `migrateShedOverrides` migration in `internal/loomengine/config.go` are removed with the keys.
  - id: F13
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md, D3 peers bullet
    summary: The peers fallback reads an empty recipe field as tapestry, but a beam project entry seeds pawl from its kind with an empty recipe field, so an off-machine pawl peer is still read against tapestry's rows
    fix: In D3's peers bullet, take the off-machine peer's recipe as Seed-Child does, pawl for a project entry and else the entry's recipe field with empty meaning tapestry, and add a pawl peer row to D3's acceptance criterion.
  - id: F14
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md, D10
    summary: A run blocked on Tapestry-Preflight by a broken batcher.yaml is mapped to Discussion-Write, so it resumes past Preflight with no batcher.yaml check until Webster, the late failure D10 exists to prevent
    fix: In D10, map a run parked on Tapestry-Preflight to Preflight rather than Discussion-Write, so the batcher.yaml load runs again on resume, or state that this map is accepted with the reason, and update D10's acceptance criterion.
  - id: F15
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md, D2
    summary: D2 models the non-loom refusal on arm.go's recipe read, but lym shed arms tapestry, darn and pawl runs through the same loomcli.ArmAt, so a refusal placed there also reaches the lym shed verbs D14 keeps for every recipe
    fix: In D2, state that the non-loom refusal sits in the `lym loom` command path and not in `loomcli.ArmAt`, which `lym shed` shares.
  - id: F16
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md, Open risks first line
    summary: The batten.yaml risk asks the parent to confirm a reading the record already settles, since the entry's Open item named only shed.yaml, loom.yaml and darn.yaml and Scope Out and D13 already exclude batten.yaml
    fix: Drop the first Open risks line; Scope Out and D13 already carry the batten.yaml exclusion.
---

### [BLOCKING] Verbatim board body breaks the record's section structure

**Location:** `_lym/discussion/decision-record.md`, D4 first paragraph
**Issue:** Board bodies carry H2 headings; this task's own body holds `## Seen`, `## Config split`, `## Open` and more.
Pasted verbatim under `## Goal`, each becomes a top-level section that `internal/discussionparser` scans for by `## ` line, and `Validate` accepts extra H2s (`internal/discussionparser/validate.go:62-64`).
A body heading named `## Decisions`, `## Open risks` or `## PATTERN changes` then receives `decision add` appends, Go carry-over blocks or the PATTERN parse in place of the record's own section.
D4's Bound covers only an empty body.
Found by walkthrough and bound-breaker.

### [MEDIUM] D10 removes the half-finished guard with no stated bound

**Location:** `_lym/discussion/decision-record.md`, D10
**Issue:** `CheckSeed`'s fresh-start rule refuses history naming any row other than the two Preflights, a set `start_sha` and `pause_requested` (`contracts/specs/loom-status-spec.md:152-153`, `internal/loomengine/coherence.go`).
D10 names only "the check that the seed is fresh for its first row and the stat race guard" and says the operator's design drops it.
The board entry's premise was that the seed check repeats Shed's own read, and D10's own research shows that holds for four verdicts only.
The discussion spec and the rubric require a Decision that removes a guard to state what it now lets through and what bounds it, and D10 states neither.
Found by bound-breaker and the handler.

### [MEDIUM] D13's refusal would be a silent drop through the existing retired-key route

**Location:** `_lym/discussion/decision-record.md`, D13
**Issue:** `RetiredKeys` keeps a key's template category; these keys are `CategoryNone` (`internal/loomengine/config.go:557-563`), which `ValidateLocal` admits and every load drops (`internal/configengine/doc.go:75-77`).
Retiring them that way makes `PrepareRunSettings` accept `loom.driver_kind` and drop it, the "silently running another" outcome D13 rules out.
Pairs seeded before landing hold the keys in their local `loom.yaml` or `darn.yaml`, which D13 does not cover.
The refusal also fires at board writes, `profiles.yaml` writes and Seed-Child (`internal/boardcli/runsettings.go:90`, `internal/battenshed/seamchild.go:150`), not "when the run starts".
Found by existing-paths, bound-breaker and interactions.

### [MEDIUM] In-flight darn runs: the halt is reached late and its way forward fails

**Location:** `_lym/discussion/decision-record.md`, D4 second paragraph
**Issue:** The `discussion` gate runs at the handoff, after a full writer session that may commit code (`internal/loomshed/darnwrite.go:67-92`).
Its exhausted Stuck carries the validator's findings plus `darnResumeWayForward`, not `goto --to Record-Write`.
The `goto` then leads through Drift-Baseline, which refuses once HEAD is past the branch point (`internal/driftengine/doc.go:29`).
Runs parked on Drift-Baseline or Drift-Await sit past Record-Write's new place and read a `decision-record` subject that does not exist.
Found by interactions, walkthrough and existing-paths.

### [MEDIUM] D6's no-commit halt is unplaced against the handoff gates and misses respawns

**Location:** `_lym/discussion/decision-record.md`, D6
**Issue:** Darn-Write's gates run `discussion`, `verify`, `description` inside the session before any producer check (`contracts/recipes/darn-recipe.yaml`).
With D4's record present the discussion gate passes, so a stop report meets the description gate, which can exhaust and halt with its own findings as the reason.
A respawn whose earlier spawn committed work has commits past the fork, so the no-commit test never fires and the run continues to Drift-Verify, which contradicts "the halt lets nothing through".
Found by walkthrough and bound-breaker.

### [MEDIUM] A no-parent answer never reaches the record that review reads

**Location:** `_lym/discussion/decision-record.md`, D6 and D7
**Issue:** Today's stencil has a no-parent writer write a `Decision for <slug>:` note's answer into the record (`contracts/stencils/darn/darn-template-write.md`, Step 3).
D6 forbids that, and `decision add` refuses a task session, so the answer stays unrecorded.
D7 makes the record the sole Code-Review answer key, so the perch judges a change that follows the note against a record that lacks it.

### [MEDIUM] Record-Write's skip test is not resume-safe

**Location:** `_lym/discussion/decision-record.md`, D4 Bound line
**Issue:** A crash after the record write but before the support log or the commit leaves a record that exists, so a resume returns done.
D5 makes Darn-Write's gate validate-only, and Darn-Write's commit covers only the landing directory (`internal/loomcli/darnwiring.go:153-154`), so the pair stays incomplete or uncommitted and the gate exhausts.

### [MEDIUM] The plan perch's hard-cap check is lost across the module split

**Location:** `_lym/discussion/decision-record.md`, D12
**Issue:** `LoadConfig` refuses a perch whose resolved hard cap is below its resolved budget, from one `Config` holding `PlanMaxBounces`, `ReviewMaxBounces` and the caps (`internal/loomengine/config.go:4`, `:446`, `:1341`).
After the split the plan keys sit in `tapestry.yaml` and their fallbacks in `loom.yaml`, and D12 does not say which load still sees both.

### [MEDIUM] Pinned specs are left out of Scope

**Location:** `_lym/discussion/decision-record.md`, Scope In
**Issue:** The discussion spec says the history check runs only on darn and that "a tapestry run's discussion check reads no history", which D5 reverses.
`contracts/specs/shed-recipe-spec.md` lists the top-level recipe keys D1 and D8 extend, and `contracts/specs/loom-status-spec.md` pins the Tapestry-Preflight row and `CheckSeed`'s verdicts D10 removes.
Scope In names docs, headers, stencils and the rubric but no spec.
`Plan-Write` reads only the record, so the specs would contradict the code after landing.

### [MEDIUM] Declared PATTERN-shed-stop-chain text contradicts the kept batten.yaml overrides

**Location:** `_lym/discussion/decision-record.md`, `## PATTERN changes`
**Issue:** The current entry ends "arbiter keys in shed.yaml with module overrides"; the declared text ends "arbiter keys in shed.yaml, a loom recipe's own through its profile block".
D13 keeps `batten.yaml`'s arbiter and tackler keys (`internal/battenshed/template.yaml`), and the landed rule would no longer admit them.

### [LOW] D13 changes a pawl path that Scope Out and D2 keep

**Location:** `_lym/discussion/decision-record.md`, D13 against Scope Out and D2
**Issue:** Pawl reads its override from `loom.yaml` through `RecipeOverride`'s non-darn branch today (`internal/loomengine/config.go:1151-1166`), and D13 moves it to `shed.yaml` "pawl included".

### [LOW] KeyCopies and migrateShedOverrides still write the retired keys

**Location:** `_lym/discussion/decision-record.md`, D13
**Issue:** `KeyCopies` copies `loom.arbiter*` into `darn.arbiter*` (`internal/configreg/keymove.go:51-57`), and loom's `migrateShedOverrides` rewrites `driver_kind` and the tackler keys (`internal/loomengine/config.go:714-778`).
Both become dead or conflicting once the keys are retired.

### [LOW] The off-machine peers fallback misreads pawl peers

**Location:** `_lym/discussion/decision-record.md`, D3 peers bullet
**Issue:** `boardRecipe` seeds pawl from `Kind == project` and leaves the `recipe` field empty (`internal/battencli/wire.go:522-527`), so "empty meaning `tapestry`" reads a pawl peer against tapestry's rows.

### [LOW] A run parked on Tapestry-Preflight skips the batcher.yaml check

**Location:** `_lym/discussion/decision-record.md`, D10
**Issue:** The retired-row map sends such a run to Discussion-Write, past Preflight, so a run blocked by a broken `batcher.yaml` resumes with no check until Webster.

### [LOW] The non-loom refusal has no stated place, and the shared arm also serves lym shed

**Location:** `_lym/discussion/decision-record.md`, D2
**Issue:** `lym shed` arms tapestry, darn and pawl through `loomcli.ArmAt` (`internal/shedcli/table.go:49-68`), the same arm whose `resolveRecipe` D2 cites as its model.

### [LOW] The batten.yaml Open risk asks the parent about a settled reading

**Location:** `_lym/discussion/decision-record.md`, `## Open risks` first line
**Issue:** The entry's Open item named `shed.yaml`, `loom.yaml` and `darn.yaml` only, and the parent's answer named `loom.yaml` and `darn.yaml`.
The writer's reading follows both, and Scope Out and D13 already record it.
It is a choice the writer made, not a question only the parent can settle, so it is not class `decision`.
No `Added after Discussion` entry or `Decision for run-verbs-shared:` note exists to settle the other risk lines, and those lines name real transition steps the orch takes, so they stand.

## Rejected

- Interactions: Record-Write's commit counts as a commit past the fork. Rejected by the lens itself, and confirmed: `_lym` is the weft repository, so discussion and landing commits never land on the warp task branch.
- Relocation (focus item 6): no relocation finding. The rationale lines that touch a rejected alternative (D13's "refusing rather than ignoring", "since moving them into shed.yaml would widen") carry the reason behind a requirement the plan writer implements, so the completeness-before-leanness test keeps them. D3's first rationale sentence is a short research trail, but it carries the facts behind D3's bullets and moving it gains nothing.

## Focus departures

None.
