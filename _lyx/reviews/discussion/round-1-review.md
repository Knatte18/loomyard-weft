---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D7 and Acceptance criteria (--amends-plan lands ... a failed commit leaves neither)
    summary: D7 promises the decision and the amendment record land together or not at all, but on a commit failure the verb today keeps the appended entry and its way forward (lyx fabric commit) then commits the decision with no amendment record
    fix: In D7, state that on a commit failure the verb also restores the decision record to its pre-append bytes (as it already does on a check finding) besides removing the amendment record, and that the commit-failure refusal's way forward becomes re-running "lyx loom decision add ... --amends-plan" instead of "lyx fabric commit"; keep the acceptance criterion "a failed commit leaves neither" and add that the record holds no new entry after it.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D9 step 1; support-log.md Turn 5 (panics carry absolute paths)
    summary: The hub's gate tiers build with -trimpath, so panic stack frames read <module path>/<dir>/<file>.go:N, a shape none of step 1's placements accepts, so every panic location is dropped
    fix: In D9 step 1, add a placement before the drop rule - a path that opens with the module path read from the worktree's go.mod followed by a slash is placed by stripping that prefix - and cite the evidence (hub gate.yaml tiers set GOFLAGS -trimpath; internal/gatecli/tiers.go trimpathLine); correct Turn 5 of the support log to say panics under -trimpath carry module-path-prefixed paths.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D9 step 3, the cards field, and the Acceptance criterion naming that card, its batch and its commit
    summary: Webster records one commit per batch (CardSHAs is set to the batch head) and the active wide batcher puts up to six cards in a batch, so no per-card commit exists for the trace to name
    fix: In D9 step 3 and the cards field, replace the card's commit with the batch's commit range, its start SHA to its recorded head from Webster's run state, and render it in PublishFailureNote as the batch range to diff; reword the acceptance criterion to name the card, its batch and that batch's commit range.
  - id: F4
    severity: MEDIUM
    class: decision
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D8 (the loom arbiter wiring puts the amendment target row into the stop envelope; a recipe that sets none)
    summary: The stop envelope is the generic shed step's output saved verbatim by the driver and loom's arbiter wiring never writes it, so D8 leaves open which layer carries the amendment target and names no recipe key for a recipe that sets none
    fix: In D8, name the carrier and its source - for example a new arbiter prompt input that loom's arbiter wiring fills from a named recipe or loom-engine setting, rendered into the arbiter stencil beside the envelope path - and state that the generic step envelope is unchanged; reword the stencil sentence and the acceptance criterion to read that input instead of the stop envelope.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D7 Rationale (The bound) and D8
    summary: Each new --amends-plan call refreshes the write time and re-admits the goto, so nothing D7 or D8 states caps repeated re-opening of Plan-Review, and the scope-widening bound is the arbiter's judgment alone while its entry becomes the Plan-Review answer key
    fix: In D8, state the bounds of the new route - repeats by the arbiter are capped by the run's arbiter spawn budget (arbiter_max_spawns, counted from attempt records on disk), repeats by the parent or orch by the spent Plan-Review bounce budget halting the judge for the parent's resume - and state that a scope widening is bounded only by the hand-on rule, since the arbiter's entry is part of the answer key Plan-Review and PR-Gate's alignment agent judge against.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D7 (removed by Plan-Bouncer's approved settle only when that settle judged a round run after the record's write time)
    summary: A second amendment recorded while the Plan-Burler-Review round runs can be removed by that round's settle without ever reaching the fixer, because D7 does not say which time of the round is compared with the record's write time
    fix: In D7, state that the record is removed only when the judged round's review started after the record's latest write time (the round's start, not its end or judge time), so an amendment appended mid-round keeps the record for one more round; add an acceptance criterion for an amendment appended while a round runs.
  - id: F7
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D7 answer-key sentence; contracts/recipes/loom-recipe.yaml Plan-Burler-Review instructions (plan-conflict clause)
    summary: D7's answer-key clause allows edits to cards Webster has not recorded done while the existing plan-conflict clause allows edits only to cards whose batch has not begun, and on the recurrence route D7 adds both bind one round with no precedence
    fix: In D7, state one rule for both clauses - for example that an amendment, like a plan conflict, edits only cards whose batch has not begun or inserts cards after the last begun batch, matching what Webster's auto-rebaseline accepts - and word the new answer-key line to the same rule.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D2 Rationale (Its bound)
    summary: D2 names only an undeclared prose edit as an escape, but a declared entry whose bullet an earlier generation already changed passes with no verbatim line, so a further prose change to it in a rework generation also escapes the plan gate's headroom projection
    fix: In D2's Rationale, extend the bound - a declared entry whose bullet already differs from the merge base and that no live card fences is projected at its current bullet, so a further prose change to it escapes the plan gate - and name the card gate's lyx loom lint-pattern and the verify gate's headroom lint as its guard, as for the undeclared edit.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D8 and its Rationale (Plan-Burler-Review is before every row that can stop on a plan defect)
    summary: The envelope offers the amendment route on every loom stop, but D7 refuses --amends-plan before the plan is approved, and Plan-Write sits before Plan-Burler-Review, so the rationale's claim is false and a pre-approval stop gets an offer the verb refuses
    fix: In D8, offer the amendment target only once the plan is approved (the same condition D7's refusal reads), and reword the rationale to say Plan-Burler-Review is before every row after plan approval that can stop on a plan defect; adjust the envelope acceptance criterion to a stop after approval.
  - id: F10
    severity: LOW
    class: decision
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D3 (Every reader of an Edit group's targets ... reads an Extends group alike)
    summary: Edit groups carry the re-sign arrow grammar (Resigns), read by the batcher's C# boundary, and D3 does not say whether an arrow under Extends is parsed, which would contradict changes nothing already there, or refused
    fix: In D3, state that a re-sign arrow sub-bullet under Extends is a blocking plan-gate finding whose way forward is to move that ref under Edit; add it to the Extends acceptance criterion.
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D9 step 3 (the overlap rule the scope guard and the batch-order check use)
    summary: Webster's batch-order check matches refs by exact string equality (refsIntersect), not by containing type, unit self glyph or file, so the attribution is false
    fix: In D9 step 3, drop and the batch-order check from the attribution, leaving the rule stated in the parenthesis as D9's own.
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D1 Rationale (the plan gate and the run-time lint cannot disagree)
    summary: With no parent recorded, D1 compares against HEAD while the verify gate with no merge base skips its headroom lint, so the two can disagree on such a run
    fix: In D1's Rationale, limit the claim to a run with a recorded parent, and state that with none the plan gate's HEAD comparison is the only plan-level headroom check.
  - id: F13
    severity: LOW
    class: decision
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D1 (replaces ... or is appended when the ID is absent; every removes entry ... deletes its ID's bullet)
    summary: D1 does not say what the projection does when the plan both removes an ID and a card fences a line for it
    fix: In D1, state the order - removes are applied after every fenced line, so a removed ID is absent from the projection - or make the pair a blocking plan-gate finding naming both the bullet and the card.
  - id: F14
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D1 way forward (ask the parent to record a changes decision ...)
    summary: On an approved plan, such as a Plan-Burler-Fix gate inside D7's route, a changes decision recorded without --amends-plan reaches no fix loop, and D1's way forward does not name the flag for that case
    fix: In D1's way forward, add that once the plan is approved the parent also passes --amends-plan with the pattern flags, then routes as D7 states.
  - id: F15
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D9 step 1 (a bare file name ... through its identity's package directory)
    summary: An opaque failure identity, the one ParseVerifyFailures yields when no go test package line parses, such as a go vet tier, carries no Package, so its bare or ./-relative locations are dropped and neither D9 nor Open risks says so
    fix: In D9 step 1, state that a bare or ./-relative location of an opaque identity is dropped into untraced, and fold the case into the Open risk on traces that name no card.
  - id: F16
    severity: LOW
    class: consistency
    origin: handler
    location: _lyx/discussion/decision-record.md Acceptance criteria against D6, D7's answer-key sentence and D8's orch stencil route
    summary: The plan-review rubric check and the plan-writer guidance (D6), the Plan-Burler-Review answer-key line (D7) and the orch stencil route (D8) have no acceptance criterion
    fix: Add acceptance criteria - the plan-review rubric names the Extends check and the plan-writer stencil says when to use Extends; the Plan-Burler-Review instruction names the amendment record; the orch stencil names --amends-plan, then lyx loom goto --to Plan-Burler-Review and lyx loom resume, and no longer allows editing a card.
  - id: F17
    severity: NIT
    class: consistency
    origin: handler
    location: _lyx/discussion/decision-record.md D5 last sentence; Open risks second bullet
    summary: D5's Decision paragraph carries its justification (the orch's 2026-10-18 estimate), and the second Open risk restates D7's last route sentence
    fix: In D5, keep the requirement that the build of D3 and D4 does not wait on a rerun in the Decision and move the clause citing the orch's 2026-10-18 estimate into the Rationale; delete the second Open risks bullet, since D7 already states the spent-budget path and the parent's resume.
---

### [BLOCKING] --amends-plan's land-together guarantee breaks on a commit failure

**Location:** `_lyx/discussion/decision-record.md` D7, and the acceptance criterion "a failed commit leaves neither".

**Issue:** `decisionVerb` (`internal/loomcli/decision.go`, the commit-failure refusal) leaves the appended entry in the working copy and tells the caller to run `lyx fabric commit`.
D7 removes only the amendment record on that failure.
Following the existing way forward commits the decision with no amendment record, and the goto is then refused.
The plan writer gets an acceptance criterion that the code path contradicts.
All four lenses found this independently.

### [MEDIUM] D9 drops every panic location under `-trimpath`

**Location:** D9 step 1; support log Turn 5.

**Issue:** The hub's tiers set `GOFLAGS: "-trimpath -p={parallel}"` (`_board/_lyx/config/gate.yaml`), and `internal/gatecli/tiers.go` `trimpathLine` documents that frames then read module-relative.
A frame like `github.com/Knatte18/loomyard/internal/x/y.go:55` does not exist as written, is not absolute, and is neither bare nor `./`-relative, so step 1 drops it.

### [MEDIUM] No per-card commit exists for the trace to name

**Location:** D9 step 3, the `cards` field and its acceptance criterion.

**Issue:** `internal/websterengine/recordbatch.go` sets `bs.CardSHAs = []string{report.HeadSHA}`, one SHA per batch.
The hub's `batcher.yaml` makes `wide` active with `max_cards: 6`.
The "card's commit" handed to the fixer as its starting point is the batch head, spanning every card in the batch.

### [MEDIUM] D8 names no carrier for the amendment target

**Location:** D8.

**Issue:** The arbiter reads `{{.envelope_path}}`, the generic step envelope that the driver saves verbatim.
`internal/loomcli/arbiter.go` `arbiterStopEnvelope` reads only `kind` and `transient`.
Loom's wiring fills the prompt, never the envelope.
"A recipe that sets none" implies a recipe setting that D8 does not name.

### [MEDIUM] Repeated re-opening and scope widening are unbounded as stated

**Location:** D7 Rationale and D8.

**Issue:** The rubric's attack-surface rule asks the new routing edge to state what bounds it.
D7's bound holds for each single goto, but every new `--amends-plan` call re-admits one.
The real caps are `arbiter_max_spawns` for the arbiter (`internal/loomcli/arbiter.go`, `MaxSpawns`) and the spent bounce budget for the parent, and D7 and D8 cite neither.
A widening is stopped only by the hand-on rule, because the arbiter's entry joins the answer key that Plan-Review and the alignment snapshot (`alignmentSnapshot`) read.

### [MEDIUM] Removal time comparison lets a mid-round amendment be lost

**Location:** D7 removal rule.

**Issue:** `decisionVerb` refuses only while Discussion-Write runs, so a parent can append amendment B while the Plan-Burler-Review round for amendment A is running.
If "a round run after the write time" compares the round's end or judge time, the approved settle removes the record, and B never reached a reviewer.

### [MEDIUM] Two answer-key clauses disagree on which cards may be edited

**Location:** D7 answer-key sentence; `contracts/recipes/loom-recipe.yaml` Plan-Burler-Review instructions.

**Issue:** The existing clause allows only cards "whose batch has not begun", and D7 allows cards "Webster has not recorded done".
At Webster's recurrence halt `.lyx/webster/plan-conflict.md` still exists (`internal/websterengine/doc.go`, the plan-conflict record), so both clauses apply to the same round.

### [MEDIUM] D2 understates what escapes the plan gate

**Location:** D2 Rationale.

**Issue:** Take a PR-Rework generation 2 that declares `changes` again on an entry generation 1 already built, with a prose instruction.
The entry passes `pattern-line-unprescribed` because its bullet differs from the merge base.
D1 then projects the old bullet, so the growth goes unmeasured at the plan gate.
The guard for this case is the card gate's `lyx loom lint-pattern` (`internal/websterengine/cardgate.go`) and the verify gate's headroom lint, and D2 does not say so.

### [LOW] The amendment route is offered before plan approval

**Location:** D8 and its Rationale.

**Issue:** D7 refuses `--amends-plan` with no approved plan.
Plan-Write precedes Plan-Burler-Review in the recipe, so a Plan-Write stop gets an offer the verb refuses, and its goto would point after the current row.

### [LOW] Re-sign arrows under `Extends` have no stated outcome

**Location:** D3.

**Issue:** `TargetGroup.Resigns` is populated only for `Edit` (`internal/planparser/plan.go`), and `csharpResignTargets` in `internal/batcher/boundary.go` reads it.

### [LOW] D9 misattributes its overlap rule

**Location:** D9 step 3.

**Issue:** `refsIntersect` in `internal/websterengine/sequence.go` compares by exact string equality.

### [LOW] Plan gate and run-time lint can disagree with no parent

**Location:** D1 Rationale.

**Issue:** `internal/loomshed/verifygate.go` skips the headroom lint when `mergeBase` is nil, while D1 then compares against `HEAD`.

### [LOW] `removes` plus a fenced line for one ID has no stated order

**Location:** D1.

**Issue:** The projection's result depends on whether the delete or the append runs last.

### [LOW] D1's way forward misses `--amends-plan` after approval

**Location:** D1 way forward.

**Issue:** A `changes` decision recorded on an approved plan without the flag reaches no fix loop, and `pattern-changes-mismatch` then fires at a later gate (`internal/loomshed/plancontext.go`, `patternChangeMismatches`).

### [LOW] Opaque identities drop bare-name locations silently

**Location:** D9 step 1.

**Issue:** `VerifyFailure.Package` is empty for the opaque identity (`internal/websterengine/verifyparse.go`).

### [LOW] Stencil-side Decisions have no acceptance criterion

**Location:** Acceptance criteria.

**Issue:** The focus asks that every Decision have a testable criterion.
D6, D7's answer-key line and D8's orch route have none, though each is a checkable text change.

### [NIT] Justification in D5's Decision paragraph; a duplicated Open risk

**Location:** D5; Open risks.

**Issue:** D5's estimate clause is rationale, not decision.
The second Open risk restates D7, which carries the requirement, so deleting the risk loses nothing the plan writer needs.

## Rejected

- D7's answer key naming `lyx loom insert-card` for the plan fixer as a denied command (`PATTERN-no-denied-recovery`): the plan-review rubric and the plan and rework stencils already direct fixers to `insert-card`, so it is not denied.
- D9 step 1 mapping an import path to a directory as a breach of `PATTERN-glyph-conversion-chokepoint`: an import path is no glyph, and the entry governs glyph-to-path conversion only.
- D4's claim that a wrong `Extends` label changes nothing today: held, since batches run in card order and change-against-change edges always run forward by card number.
- D9's quarry claim: held, quarry v0.4.0 ships `Enclose` and `EncloseAt` in `quarry/enclose.go`, and `go.mod` pins v0.4.0.
- A Decision editing the task's own board entry: none does.
